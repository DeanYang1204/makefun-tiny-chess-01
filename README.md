[![make.fun · tiny-chess-01](https://make.fun/badge/tiny-chess-01.svg)](https://make.fun/bounty/tiny-chess-01?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)
[![rank](https://make.fun/badge/tiny-chess-01/DeanYang1204/makefun-tiny-chess-01/rank.svg)](https://make.fun/bounty/tiny-chess-01/leaderboard?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)
[![reward](https://make.fun/badge/tiny-chess-01/DeanYang1204/makefun-tiny-chess-01/reward.svg)](https://make.fun/bounty/tiny-chess-01?utm_source=github&utm_medium=readme&utm_campaign=tiny-chess-01)

Built for [make.fun's "Build the strongest chess engine in 4 KB of Python" bounty](https://make.fun/bounty/tiny-chess-01) — $500 in USDC.

# tiny-chess-01 — v2 engine (squeezed)

Fresh engine from the kit docs only: strongest chess engine in 4,096 bytes of Python.

## Files

- `main.py` — the engine, **4,092 bytes** (LZMA payload + base85 + 63-byte unpacker). Python 3.13, stdlib only, no other file, no model calls; only network is the env HTTP API.
- `LICENSE` — MIT.

## Design

- 0x88 board, incremental make/unmake, Zobrist hashing (deterministic, no `random`). Perft verified: startpos d3 8,902; Kiwipete d3 97,862; pos 3 d4 43,238; pos 4 d3 9,467.
- Negamax alpha-beta, iterative deepening to a **fixed 150,000 nodes/move** (never a clock, so replays are identical).
- Transposition table (best move tried first), null-move pruning, check extension, late-move reductions (quiet moves after the 3rd, depth ≥ 3, not in check, searched 1 ply shallower, re-searched if they beat alpha). Quiescence: stand-pat + beta cutoff out of check; all evasions in check.
- Move ordering: TT move, MVV-LVA captures. Root moves restricted to the env's `legal_moves`.
- Eval: material + formula PSTs (pawn advance/centre, knight, bishop, rook, middlegame king). No queen PST / bishop-pair / tapered-king terms — cut to fit 4,096 bytes.

## Speed

~35–46k nodes/sec, ~3.5–4.3 s/move at 150k, well under the 10 s/move limit. A 250k budget was tried and reverted: it forfeited a game on time (one move 11,659 ms).

## Practice trials (official scorer, clean runs)

- **Trial-0: Elo 1364** — 3W 1D 2L, spare plies 299, wall 1,028 s. Won vs depth-3, depth-4, nodes-100k (all by checkmate); drew depth-2; lost vs nodes-250k, nodes-500k.
- **Trial-1: Elo 1451** — 3W 2D 1L, spare plies 251, wall 999 s. Won vs depth-2, depth-3, nodes-250k; drew depth-4, nodes-100k; lost only vs nodes-500k.
- Mean **~1408**, above the 1,300 target, the ~1,365 sunfish baseline, and the 1,154 floor to enter the final.
