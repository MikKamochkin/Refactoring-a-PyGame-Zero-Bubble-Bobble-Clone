# Refactoring a Pygame Zero Bubble Bobble Clone

## How to Run the Game

### 1. Install Pygame Zero
Make sure Python 3.8+ is installed.

Install Pygame Zero:
pip install pgzero

### 2. Run the Game

From the project root directory:

pgzrun cavern.py


The game should open in a window.

---

## Controls

- **Left / Right Arrow** — Move
- **Up Arrow** — Jump
- **Space** — Fire orb (hold to blow further)
- **P** — Pause / Resume (Play screen only)

---

## Architectural Changes

This project refactors the original single-state implementation into a more modular, object-oriented architecture.

### Task A - State Pattern (Screen Objects)

- Introduced an `App` class that owns the current screen.
- Replaced global state branching with screen classes:
  - `MenuScreen`
  - `PlayScreen`
  - `GameOverScreen`
- Global `update()` and `draw()` now delegate to `app.update()` and `app.draw()`.
- Screen transitions are handled via a single method: `app.change_screen(...)`.
- `Game` object creation occurs inside screen transitions instead of global logic.

---

### Task B - Input Snapshot + Edge Detection (Command Pattern)

- Removed global `space_down` and direct `keyboard` access inside `Player`.
- Introduced an immutable `InputState` dataclass.
- Built `InputState` once per frame inside `App`.
- Implemented edge detection for:
  - Starting the game (SPACE press)
  - Firing an orb (SPACE press)
  - Jumping (UP press)
  - Pausing (P press)
- `Player.update()` now consumes `InputState` instead of reading input directly.

---

### Task C - Pause Mode

- Added pause toggle using the **P** key.
- While paused:
  - Game simulation is frozen (no movement, spawns, or timers).
  - The current scene is still drawn.
  - A "PAUSED" overlay is displayed.
- Pause is only available in `PlayScreen`.
- Game resumes cleanly when P is pressed again.

---

## Notes

- Original gameplay behavior remains equivalent.
- Refactoring focuses on improved separation of concerns:
  - Input handling centralized
  - Screen management encapsulated
  - Game logic isolated from global state
