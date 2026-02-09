# DESIGN.md

## Screens Architecture (State Pattern)

The game is organized using a **Screen/State pattern** to remove global branching logic.

### Core idea
- An `App` object owns exactly one active screen: `self.screen`.
- Pygame Zero’s global `update()` and `draw()` are **thin delegates**:
  - `update()` calls `app.update()`
  - `draw()` calls `app.draw()`

### Screen objects
Each screen is a class with:
- `update(input_state)`
- `draw()`

Implemented screens:
- **MenuScreen**
  - Owns a `Game()` instance without a player (used only for background animation/level rendering).
  - On SPACE press (`fire_pressed`), transitions to `PlayScreen`.
- **PlayScreen**
  - Owns `Game(Player())` and runs the full simulation.
  - Can transition to `GameOverScreen` when the player runs out of lives.
- **GameOverScreen**
  - Reuses the existing `Game` object from `PlayScreen` so the final score/level remains visible.
  - On SPACE press, transitions back to `MenuScreen`.

### Transitions
All transitions occur through **one method**:
- `app.change_screen(new_screen)`

This centralizes screen switching and avoids scattered global state changes.

---

## Input Design (Input Snapshot + Edge Detection)

Input is handled using an immutable per-frame snapshot:

### InputState
A dataclass `InputState` is built once per frame and passed downward:
- `left: bool`
- `right: bool`
- `jump_pressed: bool` (edge: pressed this frame)
- `fire_pressed: bool` (edge: pressed this frame)
- `fire_held: bool` (level: currently held)
- `pause_pressed: bool` (edge: pressed this frame)

### Centralized construction
- `App` contains previous-frame key states (e.g., `_prev_space`, `_prev_up`, `_prev_p`).
- Each frame, `App._build_input_state()` reads current keyboard values and computes edges:
  - `pressed = now and not prev`
- `Player.update(input_state)` **never reads `keyboard.*` directly**.
- The input snapshot is passed through:
  - `App.update()` → `Screen.update(input_state)` → `Game.update(input_state)` → `Player.update(input_state)`

This removes global input flags and ensures consistent, testable control logic.

---

## Pause Mode (Task C)

Pause exists only in `PlayScreen`.

### Toggle behavior
- When `input_state.pause_pressed` is true (P edge press), `PlayScreen` toggles `self.paused`.

### While paused
- **Simulation is frozen** by skipping `self.game.update(...)`.
  - No movement, timers, spawns, enemy logic, or player updates occur.
- **Rendering continues**:
  - `draw()` still calls `game.draw()` and `draw_status()`
  - A pause overlay text (“PAUSED”, “Press P to resume”) is drawn on top.

### Resume
- Pressing P again toggles `paused` back to `False`.
- The game resumes cleanly because the `Game` object and all entities remain intact; only updates were skipped.

(Recommended restriction implemented: Pause cannot be triggered in Menu or Game Over.)
