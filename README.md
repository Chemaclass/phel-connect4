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

Requires PHP 8.5+ and Composer.

```bash
composer install
```

## Run

Use the composer scripts:

```bash
composer play           # two players (default)
composer play:2p        # same as above, explicit
composer play:1p        # vs AI; you play Red (move first), AI plays Yellow
composer play:ai-first  # vs AI; AI plays Red (moves first), you play Yellow
```

Or run directly with env vars:

```bash
./vendor/bin/phel run src/phel/main.phel                       # 2 players
AI=yellow ./vendor/bin/phel run src/phel/main.phel             # AI = yellow
AI=red AI_DEPTH=6 ./vendor/bin/phel run src/phel/main.phel     # AI first, deeper search
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
| `AI_DEPTH` | `5` | Minimax search depth. Higher = stronger and slower (`5` ≈ 2–7s, `6` ≈ 10s, `7` ≈ 35s per move) |
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

- **Bitboard representation**: two 49-bit ints encode the position (columns laid out as 7 bits each, with a separator bit to make shift-based win detection safe).
- **Win detection**: for each axis shift (vertical 1, horizontal 7, diagonal 6, diagonal 8), `pos & (pos >> s) & m & (m >> 2s)` detects a 4-in-a-row in a handful of bitwise ops instead of scanning 69 windows.
- **Negamax + alpha-beta**: center-out move ordering so good candidates prune the wide branches early.
- **Heuristic**: per-axis 2-in-a-row and 3-in-a-row counts via `popcount`, plus a center-column bonus.
- **Depth-aware scoring**: terminal wins return `win-score + depth`, so the AI prefers winning faster and losing later.
- **Opening shortcut**: the first move (empty board) plays the center column without searching.

During the AI's turn you'll see a `thinking...` line while the search runs; when it returns, the move and elapsed time are printed before the next board render. Every turn also shows a `Last: <player> → column N` line so you can see what just happened after the screen clears.
