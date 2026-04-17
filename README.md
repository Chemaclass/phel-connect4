# Connect 4 in Phel

Two-player terminal [Connect 4](https://en.wikipedia.org/wiki/Connect_Four) written in [Phel](https://phel-lang.org), a Lisp that compiles to PHP. Optional minimax AI opponent.

```
 1 2 3 4 5 6 7
|. . . . . . .|
|. . . . . . .|
|. . . . . . .|
|. . . . . . .|
|. . . O O O .|
|. . . X X X X|

Red (X) wins!
```

## Install

Requires PHP 8.3+ and Composer.

```bash
composer install
```

## Run

### Human vs human

```bash
./vendor/bin/phel run src/phel/main.phel
```

### Human vs AI

```bash
AI=yellow ./vendor/bin/phel run src/phel/main.phel   # AI plays yellow
AI=red    ./vendor/bin/phel run src/phel/main.phel   # AI plays red (moves first)
```

## How to play

- Red (`X`) moves first, then players alternate.
- Drop a token into any column that is not full; it falls to the lowest empty row.
- First player to line up **four in a row** (horizontal, vertical, or diagonal) wins.
- Board full with no winner → draw.

### Commands at the prompt

| Input | Action |
|-------|--------|
| `1`..`7` | Drop token into that column |
| `u` | Undo the last move (replays history from scratch) |
| `q` or `quit` | Abort game |

## Configuration

Environment variables:

| Variable | Default | Meaning |
|----------|---------|---------|
| `AI` | *(off)* | `red` or `yellow` — which side the AI plays |
| `AI_DEPTH` | `3` | Minimax search depth. Higher = stronger and slower (`4` ≈ 30–60s per move, `5`+ is painful) |
| `COLOR` | *(on)* | Set to `0` to disable ANSI colors and screen clearing |

Example: strong AI, no colors, piped input.

```bash
COLOR=0 AI=yellow AI_DEPTH=4 ./vendor/bin/phel run src/phel/main.phel
```

## Tests

```bash
./vendor/bin/phel test
```

Covers board logic, game state transitions, and AI tactics (forced win, forced block, center preference).

## Project layout

```
src/phel/
  board.phel    ; pure board ops: make/drop/winner?/valid-cols
  game.phel     ; TGame struct: step, undo, game-over?
  render.phel   ; ANSI rendering + display names
  ai.phel       ; negamax with alpha-beta pruning
  main.phel     ; CLI loop

tests/phel/
  board_test.phel
  game_test.phel
  ai_test.phel
```

## AI notes

- Negamax with alpha-beta pruning.
- Move ordering: center-out (center columns dominate 4-in-a-row lines).
- Heuristic: sliding 4-window score + center-column bonus. Terminal wins score `win-score + depth` so shorter wins are preferred.
- Opening shortcut: an empty board plays the center column without searching (symmetric root would waste seconds).

Strength scales with `AI_DEPTH`, but PHP is slow — depth 3 is the sweet spot for interactive play. Depth 5+ finds tactical wins but blocks the terminal for minutes.
