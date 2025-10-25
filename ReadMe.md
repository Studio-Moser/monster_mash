# Monster Board Game

A simple strategy puzzle game where players place and manipulate monster characters on a 5x5 board to eat each other and score points. The objective is to delay a full board (Game Over) for as long as possible while maximizing score.

---

## 🎮 Gameplay Overview

* **Board:** 5x5 grid (25 spaces)
* **Goal:** Prevent the board from filling up
* **Scoring:** +1 point for each character eaten

### 🎲 Pieces

#### Characters

* Types: 🔴 Red, 🔵 Blue, 🟢 Green, 🟡 Yellow
* Each has a direction: ↑, ↓, ←, →
* Eating Rules:

  * 🔴 eats 🔵
  * 🔵 eats 🟢
  * 🟢 eats 🟡
  * 🟡 eats 🔴

#### Action Pieces

* Can only be placed **on top of a character**
* Cannot be presented if there are **no characters** on the board
* Actions:

  * `↻` Rotate 90° clockwise
  * `↺` Rotate 90° counter-clockwise
  * `↑ ↓ ← →` Move one space in the given direction

---

## 📋 Rules

1. A random piece is presented to the player (character or action)
2. The player places the piece on the board:

   * Characters must go on empty spaces
   * Actions must go on existing characters
3. If a character faces a neighbor it can eat, it moves into that space and consumes the character
4. Chain eating is allowed (recursively)
5. The board is updated and a new piece is presented
6. Game ends when the board is full

---

## 🧠 Strategy Tips

* Watch the direction of characters
* Place characters where they can eat or avoid being eaten
* Use action pieces to manipulate direction and position

---

## 🛠 Prototype Tech Stack

* HTML / CSS / JavaScript (Vanilla)
* No frameworks or backend dependencies

---

## 🚧 Future Enhancements

* Add animations and sound
* Mobile responsiveness
* Leaderboard or high score persistence
* Multiple game modes

---

## 📁 How to Run

1. Clone the repo or download the HTML file
2. Open `index.html` in any modern browser (Chrome, Firefox, Safari)
3. Play the game!

---

## 👨‍💻 Development Notes

* Game state is managed entirely in-memory
* UI updates triggered on every placement
* Logic is structured for readability and extendability

Happy monster munching! 🧟‍♂️🍴
