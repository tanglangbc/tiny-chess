# tiny-chess-01 — a 4 KB chess engine

[![make.fun · tiny-chess-01](https://make.fun/badge/tiny-chess-01.svg)](https://make.fun/bounty/tiny-chess-01?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)
[![rank](https://make.fun/badge/tiny-chess-01/tanglangbc/tiny-chess/rank.svg)](https://make.fun/bounty/tiny-chess-01/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)
[![reward](https://make.fun/badge/tiny-chess-01/tanglangbc/tiny-chess/reward.svg)](https://make.fun/bounty/tiny-chess-01?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)

Built for [make.fun](https://make.fun/?ref=NVRADX) — entry for [tiny-chess-01](https://make.fun/bounty/tiny-chess-01).

An original, dense Python chess engine that fits in 4,096 bytes as
committed. It plays through the make.fun env (`MAKEFUN_ENV_URL`) and
picks moves by search, never by clock: every move is derived from the
position, the game moves and `MAKEFUN_TRIAL_SEED` only, so a replay
produces identical moves.

## Engine

- **Board:** 0x88, full legal move generation (castling, en passant,
  promotion, check detection). The root move list is intersected with
  the env's `legal_moves` as a safety net.
- **Search:** negamax alpha-beta with iterative deepening to a fixed
  node budget (110,000 nodes/move), transposition table (Zobrist,
  seeded `random.Random(0)`), move ordering (TT move, captures by
  MVV-LVA, killers, history), quiescence over captures/promotions and
  replies to check (with delta pruning), null-move pruning, late-move
  reductions (deeper for late moves) and check extensions. Repetition and fifty-move score 0.
- **Evaluation:** material (P 100, N 320, B 330, R 500, Q 900) +
  piece-square tables + bishop pair + endgame king centralisation
  (when little non-pawn material remains).
- **Determinism:** no wall clock, no unseeded randomness. The node
  budget — not time — ends the search, so live and replay agree.

## Size

`main.py` is a one-line packed entrypoint: the engine source is
packed with the kit's `makefun compress` (an lzma payload in a small
stub), then unpacked and `exec`'d at runtime (explicitly allowed by
`RULES.txt`; the house sunfish baseline is shipped the same way).
Packed size is 3,319 bytes, under the 4,096-byte cap. The unpacked
source is kept as `main_v4b.py` in the repository's folder and is not
committed, per the kit's flow. No other file is read, imported or run.

## Practice results (local kit, 6 games each)

| instance | Elo | W/D/L |
|----------|-----|-------|
| trial-0  | 1,451 | 4/0/2 |
| trial-1  | 1,364 | 3/1/2 |
| trial-2  | 1,279 | 2/2/2 |
| **mean** | **1,365** | 9/3/6 |

Sunfish baseline on the same 6-game practice trials: 1,364 / 1,451 /
1,279, mean 1,365 — this engine ties the baseline on the mean.

Floor to enter the final is 1,154. Per-move time averages ~2.04 s
(max ~2.73 s), well inside the 10 s move limit and the 1,800 s
trial wall clock.

## Licence

MIT. See `LICENSE`.
