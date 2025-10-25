# Monster Mash – Godot Technical Architecture

This technical architecture outlines how to implement the **Monster Board Game** using the **Godot 4** engine. It translates the high-level strategy into a concrete project structure, modules, and systems, optimized for an AI-centric workflow and portrait 2-D production.

**Document Status:** ✅ Updated for Godot 4.3+ compatibility  
**Last Review:** 2025-10-25

---

## 1) Project Overview

**Engine:** Godot 4.3+ (pin to 4.3 or 4.4 for stability)  
**Primary language:** GDScript 2.0  
**Platforms (initial):** iOS, Android, Desktop (Steam)  
**Orientation:** Portrait  
**Rendering:** 2-D (Compatibility renderer for broad device support)  
**Monetization:** Cosmetic themes, Tip Jar, rewarded ads (no gameplay advantage)  
**Video capture:** OS-level APIs (ReplayKit / Android MediaProjection) or plugin wrapper  
**Share flow:** Short highlight capture (+ optional selfie overlay), social share intent

**Key Godot 4 Features Used:**
- Signal syntax: `signal_name.emit()` (not `emit_signal()`)
- Typed GDScript with `@export` annotations
- `y_sort_enabled` for 2D depth sorting (YSort removed in Godot 4)
- `ConfigFile` for settings persistence
- `AnimationPlayer` + `Tween` for all animations

---

## 2) High-Level Architecture

| System | Purpose |
|---|---|
| Board System | 5×5 grid, placement rules, chain resolution |
| Piece System | Monster (color, facing), Action (rotate/move) |
| Eat/Combo System | RPS loop, chain detection, scoring |
| Animation System | Keyframed placement, lunge/chomp, combo |
| UI System | Menus, HUD, Game Over, Store, Tip Jar |
| Share System | Ring buffer, clip export, selfie overlay |
| Monetization | IAP (themes, tip jar), rewarded ads |
| Analytics | Events, retention metrics, tuning |
| Save/Data | Local profile, purchases, settings |

---

## 3) Repository & Project Layout

```
/project
  /scenes
    Main.tscn
    Menu.tscn
    Game.tscn
    GameOver.tscn
    Store.tscn
    TipJar.tscn
    Settings.tscn
    ShareOverlay.tscn
    /board
      Board.tscn
      Cell.tscn
  /scripts
    Main.gd
    Game.gd
    /board
      BoardLogic.gd
      Cell.gd
      EatRules.gd
    /pieces
      Monster.gd
      ActionPiece.gd
      NextPieceGenerator.gd
    /systems
      ScoreSystem.gd
      ComboSystem.gd
      AnimationSystem.gd
      ShareSystem.gd
      Monetization.gd
      IAPManager.gd
      AdsManager.gd
      Analytics.gd
      UserData.gd
    /theme
      GameTheme.gd
      ThemeManager.gd
    /util
      RNG.gd
  /themes
    themes_manifest.json
    classic.tres
    neon_future.tres
    /neon_future
      board.png
      monsters/
      sounds/
  /ui
    hud/
    styles/
  /audio
  /art
  /export
```

**Autoload singletons (Project Settings → AutoLoad):**
- `UserData.gd`, `Monetization.gd`, `IAPManager.gd`, `AdsManager.gd`, `Analytics.gd`, `ShareSystem.gd`, `ThemeManager.gd`, `RNG.gd`

---

## 4) Scene & Node Hierarchy

**Main.tscn** — scene router (Menu ⇄ Game ⇄ Store, etc.)

**Game.tscn**
```
Game (Node)
 ├─ Board (Node2D)
 │   ├─ Grid (Node2D)  # Container for Cell instances
 │   ├─ Monsters (Node2D)  # Set y_sort_enabled = true for depth sorting
 │   ├─ AnimationPlayer
 │   └─ BoardLogic.gd
 ├─ HUD (CanvasLayer)
 │   ├─ ScoreLabel (Label)
 │   ├─ NextPiecePreview (Control)
 │   └─ ShareButton / ReplayButton (Button)
 ├─ ComboOverlay (CanvasLayer)
 └─ Game.gd
```

---

## 5) Core Logic Modules

### 5.1 Board System (`BoardLogic.gd`)
- Maintains `grid: Array` (flat array with index math: `idx = y * BOARD_SIZE.x + x`) for 5×5 cells.
- Each cell stores a reference to a `Monster` node or `null` if empty.
- Validates placement rules:
  - Monsters place on empty cells.
  - Actions place on top of an existing monster.
- Emits signals:
  - `piece_placed(piece: Node, pos: Vector2i)`
  - `eat_triggered(attacker: Node, victim: Node, pos: Vector2i)`
  - `combo_resolved(chain_len: int)`
- Public API:
  - `can_place_piece(piece: Node, pos: Vector2i) -> bool`
  - `place_piece(piece: Node, pos: Vector2i) -> void`
  - `resolve_after_placement() -> void`
  - `world_to_cell(world_pos: Vector2) -> Vector2i` (maps input to grid)
  
**Grid Implementation Note:**  
For a fixed 5×5 board, use a simple flat `Array` rather than nested arrays or TileMap. This simplifies access patterns and reduces indirection. Cell visual nodes can be instantiated separately in the Grid container.

### 5.2 Piece System
- **Monster.gd** (`Node2D`):
  ```gdscript
  enum Color { RED, BLUE, GREEN, YELLOW }
  enum Dir { UP, DOWN, LEFT, RIGHT }

  @export var color: Color
  @export var dir: Dir
  var is_animating := false
  ```
- **ActionPiece.gd** (`Resource` or `Node2D` if visualized):
  ```gdscript
  enum ActionType { ROTATE_CW, ROTATE_CCW, MOVE_UP, MOVE_DOWN, MOVE_LEFT, MOVE_RIGHT }
  @export var type: ActionType
  ```
- **NextPieceGenerator.gd**: weighted random for next preview, ensures actions only appear when ≥1 monster on board.

### 5.3 Eat/Combo System
- **EatRules.gd** contains the RPS mapping:
  ```gdscript
  # res://scripts/board/EatRules.gd
  class_name EatRules
  
  enum Color { RED, BLUE, GREEN, YELLOW }
  
  const EATS := {
      Color.RED: Color.BLUE,
      Color.BLUE: Color.GREEN,
      Color.GREEN: Color.YELLOW,
      Color.YELLOW: Color.RED
  }
  
  static func can_eat(attacker_color: Color, victim_color: Color) -> bool:
      return EATS.get(attacker_color) == victim_color
  ```
- Algorithm:
  1. After placement or action resolution, scan neighbors for facing matches.
  2. Build an ordered list of eat steps (attacker → victim) using a deterministic resolver.
  3. Animate sequentially; update grid each step.
  4. Accumulate chain length; emit `combo_resolved(chain_len)`.

**Deterministic resolution order (ties):**
- Start from the last-placed or last-moved attacker.
- Direction priority: Up, Right, Down, Left.
- Then position priority: row-major (y, then x).

### 5.4 Scoring
- **ScoreSystem.gd**:
  - `on_eat_triggered` → `score += 1`.
  - Chain bonus is inherent: each link is one point.
  - Signals `score_changed(new_score)` to HUD.

### 5.5 Input Strategy (Mobile-friendly)
- Use `_input(event)` in `BoardLogic.gd` for direct board touch handling (has priority).
- Convert touch/mouse position to board coords via `world_to_cell()`:
  ```gdscript
  func _input(event: InputEvent) -> void:
      if event is InputEventScreenTouch and event.pressed:
          var cell_pos = world_to_cell(event.position)
          if is_valid_cell(cell_pos):
              handle_cell_click(cell_pos)
  ```
- For a 5×5 board, math-based hit testing is simpler and more efficient than 25 `Area2D` nodes.

### 5.6 Determinism & RNG
- Use a dedicated `RandomNumberGenerator` instance in `RNG.gd` (autoload) or within `UserData`.
- Seed at game start and log the seed with inputs to enable deterministic replays.
- Avoid global `randi()`/`randf()`; always call through the RNG instance.

---

## 6) Animation System

- Per-monster **AnimationPlayer** clips:
  - `Place`, `Idle`, `Lunge`, `Chomp`, `Celebrate`
- Use **AnimationTree** when blending between `Idle` and transient actions.
- Tracks:
  - Transform (position/rotation/scale), sprite frame, shader params.
- Combo flair:
  - Camera shakes (subtle), scale pops, screen-space fun effects in `ComboOverlay`.
- All timings centralized in `AnimationSystem.gd` constants for tuning.

- For short translations/rotations, prefer `Tween` for lower overhead; reserve `AnimationPlayer` for multi-track sequences.
- Pool monster nodes and reuse tweens to minimize allocations.

**Example lunge → chomp sequence (pseudo-GDScript):**
```gdscript
await monster.play_anim("Lunge")
await victim.play_anim("Chomp")
board.swap_cells(attacker_pos, victim_pos)
await board.animation_player.animation_finished
```

---

## 7) UI & UX

**Menu**
- Buttons: Play, Themes, Tip Jar, Settings.

**HUD**
- Score, Next Piece, Share/Replay buttons.
- Respect safe areas (notches) using anchors/margins; keep under a `CanvasLayer`.
- Use `DisplayServer.get_display_safe_area()` on mobile to avoid notches:
  ```gdscript
  var safe_rect = DisplayServer.get_display_safe_area()
  hud_top_margin.size.y = safe_rect.position.y
  ```

**Game Over**
- Score summary, quick share CTA, retry.

**Store / Tip Jar**
- Reads `themes_manifest.json`.
- Uses `Monetization.gd` → `IAPManager.gd` for purchase flows.

---

## 8) Share System

**Goal:** Capture ~5 seconds prior to a combo (chain ≥ 2) and export a short, vertical clip with optional selfie overlay.

- **ShareSystem.gd** (autoload):
  - Manages a ring buffer of frames or events.
  - On `combo_resolved(chain_len >= 2)`:
    - Collect recent frames or rebuild deterministic replay.
    - If selfie enabled, add PIP overlay.
- **Platform bridges**:
  - iOS: ReplayKit.
  - Android: MediaProjection plugin.
- **ShareOverlay.tscn**:
  - Preview + platform buttons (TikTok, IG, Twitter).

**Deterministic replay (recommended):**
- Log inputs + RNG seeds.
- Re-simulate N seconds at turbo speed; render to texture; encode.

---

## 9) Monetization

- `Monetization.gd` singleton with `purchase_theme()`, `open_tip_jar()`, `show_rewarded_ad()`  
- Cosmetic themes are Resource-based (`GameTheme.tres`) for editor previews; JSON is used only for store metadata and remote manifests.  
- Tip Jar = fixed tiers (e.g., 0.99, 2.99).  
- Rewarded ads optional for theme previews.  
- Integrations: Godot Android/iOS templates or GDExtension wrappers.

---

## 10) Analytics

**Analytics.gd**
- Tracks: session start/end, eat, combo, share, IAP, ad events.  
- Lightweight: Firebase plugin or self-hosted endpoint.

---

## 11) Save & Data

**UserData.gd**
- Stores high score, themes, settings.  
- Use `ConfigFile` (.cfg format) in `user://settings.cfg` for simplicity and built-in validation.  
- Optional cloud sync.

---

## 12) Asset Pipeline

| Asset | Format | Notes |
|---|---|---|
| Sprites | PNG / Atlas | Export vector sources to fixed sizes |
| Animations | AnimationPlayer clips | Stored as .tres for diffs |
| Audio | WAV → OGG | Loop points for music |
| UI | .tscn + Theme | Consistent portrait scaling |

---

## 12a) Theme System & Editor Previews

- Themes are `Resource` assets for live editor previews.
- `GameTheme.gd` defines exported fields for textures, audio, and colors.
- `ThemeManager.gd` (autoload) exposes `current_theme` and signals changes to interested nodes.

```gdscript
# res://scripts/theme/GameTheme.gd
@tool
extends Resource
class_name GameTheme

@export var board_texture: Texture2D
@export var monster_sprites: Dictionary  # color enum -> Texture2D
@export var arrow_style: Texture2D
@export var sfx_place: AudioStream
@export var sfx_eat: AudioStream
@export var combo_vfx_color: Color
```

```gdscript
# res://scripts/theme/ThemeManager.gd
extends Node

signal theme_changed(theme: GameTheme)

var current_theme: GameTheme

func set_theme(theme: GameTheme) -> void:
    current_theme = theme
    theme_changed.emit(theme)
```

---

## 13) Build & Deployment

- Mobile: portrait only; request camera/mic for selfie overlay.  
- Desktop: optional leaderboards via Steamworks.  
- Web: demo-only; selfie disabled.  
- CI: Godot headless export, `gdformat`, `gdlint`, and GUT tests.

**Local developer commands (Cursor-friendly):**

```makefile
# Makefile
test:
	godot --headless --path . -s res://addons/gut/gut_cmdln.gd -gdir=res://tests -gexit
format:
	gdformat .
lint:
	gdlint .
export-linux:
	godot --headless --path . --export-release "Linux" export/linux/Monster.x86_64
```

**GitHub Actions (example):**

```yaml
name: Godot CI
on: [push, pull_request]
jobs:
  build-test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: chickensoft-games/setup-godot@v1
        with:
          version: 4.3
          use-dotnet: false
      - name: Install gdtoolkit
        run: pipx install gdtoolkit==4.3.2
      - name: Lint/format
        run: gdlint . && gdformat --check .
      - name: Run GUT tests
        run: godot --headless --path . -s res://addons/gut/gut_cmdln.gd -gdir=res://tests -gexit
      - name: Export Linux
        run: godot --headless --path . --export-release "Linux" export/linux/Monster.x86_64
```

---

## 14) Performance

- Small grid = minimal load.  
- Prefer replay over video buffer.  
- **Node Pooling**: Create a pool of Monster nodes at startup; reuse via `hide()` / `show()` instead of `queue_free()` / `instantiate()`.
  ```gdscript
  # MonsterPool.gd
  var available_monsters: Array[Monster] = []
  
  func get_monster() -> Monster:
      if available_monsters.is_empty():
          return Monster.new()
      return available_monsters.pop_back()
  
  func return_monster(m: Monster) -> void:
      m.hide()
      available_monsters.append(m)
  ```
- Sprite atlases minimize draw calls (use single texture for all monsters).
- Batch animations: wait for multiple eats to finish before allowing next input.

---

## 15) Error Handling & Recovery

**Save/Load Failures:**
```gdscript
func load_user_data() -> void:
    var config = ConfigFile.new()
    var err = config.load("user://settings.cfg")
    if err != OK:
        push_warning("Save file corrupted or missing, using defaults")
        set_default_settings()
        return
    # ... load settings
```

**IAP/Ads Failures:**
- Monetization calls should fail gracefully; never block gameplay
- Log failures to Analytics for debugging
- Show user-friendly messages: "Store unavailable, try again later"

**Network Failures:**
- Analytics and share features are optional; queue events locally if offline
- Retry with exponential backoff

---

## 16) Security & Privacy

- Request camera/mic just-in-time.  
- Store only non-PII analytics.  
- Rely on platform validation for IAP.

---

## 17) Testing Strategy

- Unit tests: placement, eat, chain logic.  
- Integration: animation sync, combo timing.  
- UX: portrait ergonomics.  
- Platform: share/export QA.  
- Monetization: sandbox validation.

---

## 18) Example Code

(See earlier GDScript snippets for EatRules & BoardLogic.)

---

## 19) AI Workflow

- Use text resources for all modifiable assets.  
- AI tools can tune constants, generate JSON themes, etc.  
- Platform bridges remain thin, declarative wrappers.

---

## 20) Roadmap

1. Core loop  
2. Animations  
3. Share MVP  
4. Store/IAP  
5. Rewarded Ads  
6. Analytics  
7. Platform QA  

---

## 21) Project Settings

- `display/window/handheld/orientation = "portrait"`
- Use `stretch/aspect = keep_height` (portrait) and design around a fixed logical height.
- Enable only required permissions; request camera/mic just-in-time in share flow.

---

**End of File.**
