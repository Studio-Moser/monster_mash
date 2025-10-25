# Monster Board Game: Full Strategy, Mechanics, and Monetization Document

## Overview
"Monster Board Game" is a casual yet strategic puzzle game that centers around spatial logic, color-eating mechanics, and tactical use of actions. Eventually to be released as mobile app on iOS and Android as well as desktop game for Steam. Additionally adding to Nintendo Switch would be great.

The game is designed to be elegant, strategic, and shareable. With no gameplay-altering purchases, its monetization aligns with modern expectations for fairness. By focusing on gameplay-first design, strong visual identity, and powerful social sharing tools, it is positioned for strong organic growth and sustainable revenue.

---

## Vision and Philosophy

**Core Objective**: Create a fair, strategic, and visually satisfying game experience with viral shareability potential. The game must:
- Deliver **great core mechanics** with deep replayability.
- Emphasize **aesthetic appeal and polish**.
- Foster **spontaneous, shareable moments** (especially via video).
- Avoid pay-to-win mechanics and respect gameplay integrity.

---

## Game Mechanics

### 1. Game Board
- 5x5 grid of cells (25 spaces total).
- Each cell can be empty, hold a character piece, or an action piece.

### 2. Character Pieces
- Four monster types by color:
  - 🔴 Red
  - 🔵 Blue
  - 🟢 Green
  - 🟡 Yellow
- Each character has a facing direction: Up (↑), Down (↓), Left (←), Right (→).
- On placement, character is randomly assigned a direction.

### 3. Eating Rules
- Eating occurs if a character is facing a valid adjacent opponent based on this loop:
  - Red (🔴) eats Blue (🔵)
  - Blue (🔵) eats Green (🟢)
  - Green (🟢) eats Yellow (🟡)
  - Yellow (🟡) eats Red (🔴)
- Eating causes the attacker to move into the victim's cell, removing the victim.
- Eating can trigger a chain reaction (multi-eat combo).
- Each successful eat = +1 point.

### 4. Action Pieces
- Include:
  - Rotate clockwise (↻)
  - Rotate counter-clockwise (↺)
  - Move up/down/left/right (via arrow)
- Can only be placed **on top of a character piece**.
- Only generated when **at least one character** exists on the board.
- After action is applied, check for new possible eats.

### 5. Piece Flow
- At all times, player sees a "Next Piece" preview.
- Players tap a grid cell to place that piece.
  - If it's a character, must place on an empty cell.
  - If it's an action, must place on a character.
- After placement, system automatically checks for valid eating events.

### 6. Game Over
- Triggered when board is fully occupied and no further moves are legal.
- Game Over screen shows score, top scores, and sharing options.

---

## Shareability and Viral Moments

### 1. Combo Highlights
- If a character performs a chain of 2+ eats in one move:
  - Trigger "Highlight Moment" logic.
  - System captures last ~5 seconds of gameplay.
  - Prompts user to preview/save/share clip.

### 2. Selfie Overlay (iOS/Android)
- Optional front camera reaction overlay during highlight.
- On iOS: Explore integration with Apple’s selfie reaction API.
- Result: vertical video with gameplay + user expression.

### 3. Social Sharing
- Clip share buttons (TikTok, IG Reels, Twitter, etc.)
- Share includes watermark and branded hashtag (#MonsterBoardCombo).
- Shared videos drive organic installs.

---

## Monetization Strategy

**Principle**: Monetize through aesthetic and optional content only. Never alter gameplay through payments.

### 1. Cosmetic Theme Packs (IAP)
- Board background skins
- Monster visual styles
- Arrow direction styles
- Examples:
  - Neon Future
  - Underwater Lagoon
  - Halloween Pack
- Can be seasonal or event-driven

### 2. Tip Jar (IAP)
- Support-the-dev button
- Multiple tiers (e.g., $0.99, $2.99)
- Each tier may unlock small visual bonus (e.g., sparkly arrows)

### 3. Rewarded Ads
- Only available as optional UX:
  - Watch to preview themes
  - Watch to unlock time-limited visual perks
- No bonus pieces, moves, or scoring advantages

---

## Aesthetic Direction

### Art & Animations
- Monsters are fun, slightly grotesque cartoons
- Arrows bold and color-themed
- Eating has a lunge + chomp animation
- Chain eats show bounce/zoom motion

### Sound Design
- Plop sound for placing pieces
- Slurp/chomp for eats
- Multi-eat combo fanfare
- Subtle background loop (theme-dependent)

### Themes
- Each theme includes:
  - Board visual skin
  - Monster design variant
  - Sound/VFX variant

---

## App Structure & UI/UX

### Main Screens
1. Splash / Loading
2. Main Menu
   - Play
   - Themes Store
   - Tip Jar
   - Settings
3. Game Screen
4. Game Over Screen
5. Share Clip Overlay
6. Theme Store
7. Tip Jar UI

### Core User Flow
- Main Menu → Game Screen
- During game: place piece → eat logic → score updates
- Combo triggers Share Moment Overlay
- Game ends → Game Over screen
- Return to menu or replay

---

## Analytics & KPIs

| Metric                         | Target                |
|-------------------------------|------------------------|
| Day 1 Retention               | > 40%                 |
| Day 7 Retention               | > 20%                 |
| Avg. Session Length           | ~5+ mins              |
| Combo Clip Trigger Rate       | > 30% of sessions     |
| Share Clip Rate               | > 10% of players      |
| IAP Conversion (Themes/Tip)   | ~2% of DAUs           |
| Ad Completion Rate (Rewarded) | > 70%                 |

---

## Development Roadmap

### Phase 1: Core Gameplay
- Build game board engine (HTML/JS prototype → Mobile engine)
- Piece logic, placement, eats, scoring
- Action logic
- Game Over detection

### Phase 2: Aesthetic Layer
- Monster art + theme variants
- Sound FX
- Basic animations (placement, eat, combo)
- Polish for first public beta

### Phase 3: Share Feature & Social
- Highlight detection logic
- Auto-capture clip buffer
- Selfie overlay
- Share SDKs
- Analytics integration

### Phase 4: Monetization 
- IAP themes & Tip Jar
- Optional rewarded video system
- Theme store implementation
- Analytics tuning

### Phase 5: Launch + LiveOps
- Soft launch (test region)
- Retention + feedback loop
- Bug fixes + A/B tests
- Global launch + marketing
- Post-launch: New themes, events

---

## Research-Backed Rationale

- NYT Games (Wordle, Letterboxed): Monetized via subscriptions + theme/crossword access. Gameplay remains free.
- Indie games like Threes: One-time purchase or light IAP with zero pay-to-win.
- Games like Lex: Emphasize beauty and clarity over monetization.
- TikTok clips + selfie overlays: High-performing content for game marketing.
- Users respond to fair games with cosmetic-only monetization (source: GameRefinery, Udonis, Clevertap).
