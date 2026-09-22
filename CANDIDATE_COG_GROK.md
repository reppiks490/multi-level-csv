# Candidate cog — Grok (xAI) — 2026-09-22

Six batches on `reppiks490/multi-level-csv` (commits 17:32–17:48Z).
Do not dump these into `history/NQ_1m.csv`. They are not the execution tape.

## Batches (177 CSVs, 0 filename overlap)

| Zip | Files | Names |
|---|---|---|
| Csv first 60.zip | 30 | AAPL, MSFT, MAG7 |
| First 60 half.zip | 30 | AAPL, GOOGL, JNJ, TSLA |
| Csv 2nd 60.zip | 30 | INTC, JNJ, PFE, TSMC |
| 2nd 60 half.zip | 30 | GOOGL, MSFT, TSLA, TSMC |
| Csv last 57.zip | 29 | AMD, BRK.B, TSLA, XOM |
| Last 57 half.zip | 28 | AMD, CAT, JPM, ORCL |

All headers: `time,open,high,low,close,RATE ST,MP POC,MP VAH,MP VAL,Long,Short,TIDE Long,TIDE Short,L_TP*,S_TP*,SL`.
That is a **stock candle + Tide overlay**. Empty POC is common.

## Role of each name (no NQ default)

| Cluster future (execution) | Candidates (sensors) | Why this cluster |
|---|---|---|
| NQ | AAPL MSFT GOOGL TSLA AMD ORCL INTC MAG7 TSMC | Nasdaq-weight / semis / MAG7 basket |
| ES | JNJ PFE BRK.B XOM JPM | S&P breadth: health, energy, financials, Berkshire |
| YM | CAT | industrials / Dow component |

Execution futures stay `history/{NQ,ES,YM,GC,…}_{tf}.csv` via `icarus-plant ingest-drop`.
Candidates stay out of `history/drop/` (`BATS_*`, `LSE_DLY_MAG7`, `BCBA_DLY_TSMC`).

## How the engine learns

1. **Opus — candidate audit now**  
   Align candidate close onto the cluster future clock (forward-fill, do not invent bars).  
   Score sign agreement vs future, Tide vs Pulse side, RS vs cluster mean.  
   Reject if overlap < 200 bars or Tide is all-zero.

2. **Astra — ML later**  
   Features = residuals + Tide flags + MP distance when non-null.  
   Labels = Pulse/emulator on the **future**, not the stock.  
   Train per cluster. Do not mix Renko into this join. Do not size AAPL as NQ.

3. **Adaptation**  
   If NQ-cluster agreement dies while ES-cluster holds, that is breadth, not a broken NQ model.

## Precision

- `60` and `61` are different clocks. Do not merge.
- `1` / `1 2` / `1 3` are export variants. Dedupe by hash.
- `1M` is monthly, not 1-minute.
- Tide is a signal, not a label. QQQ is not NQ. AAPL is not NQ.

Grok (xAI). Do not rewrite Pulse to hook these files.
