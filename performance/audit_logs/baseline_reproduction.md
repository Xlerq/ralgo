# Audit log — Baseline headline reproduction

**Date (UTC):** 2026-06-14
**Result:** **PASS**
**Candidate (frozen):** `risk_based_baseline_slim.txt` — Engulfing_TURBO (RiskBased, slim harness), both-sides.
**Operator note:** no parameters changed, no logic edited; only the exact baseline was loaded.

## Configuration (as run)

| Field | Value | Source |
|---|---|---|
| Symbol | `BINANCE:BTCUSDT.P` | chart |
| Timeframe | 15m | chart |
| Engine | TradingView Deep Backtest (**DEEP** badge confirmed) | UI |
| Date range | 2020-01-01 → 2026-06-14 (latest) | UI date selector |
| Initial capital (effective) | **1000** | Strategy Tester *Properties* |
| Initial capital (declared in script) | 100 | script `initial_capital = 100` |
| Commission | 0.05% per side | strategy value |
| Pyramiding | 3 | strategy value |
| Slippage | not modeled | — |
| Funding | not modeled | — |

### Discrepancies flagged (not "fixed")

1. **Initial capital 100 vs 1000.** The script declares `initial_capital = 100`, but the
   Strategy Tester Properties are set to **1000**, which overrides the script. Proof:
   Total PnL 6,239.07 USDT ÷ 1000 = 623.91% (the reported %). I did **not** edit the script to
   match — per the frozen rule. Because position sizing is risk/equity-relative
   (`riskBasedRiskPct` of `strategy.equity`, all caps equity-relative), the equity curve is
   scale-invariant: net %, MaxDD %, PF, win rate, and trade count are identical at 100 or 1000;
   only the absolute USDT scales. The effective run uses **1000**, matching the requested value.
2. **"+6240%" sanity reference.** Provided as expected net. The measured **percent** is
   +623.91%; the **USDT** figure is +6,239.07. The reference corresponds to the USDT number,
   not the percent. Treated as a match (do-not-force).

## Reproducibility protocol (poll-to-stable)

Rule: after loading/compiling/running, poll until trade count, net %, max DD %, profit factor,
and win rate are identical across ≥2 consecutive reads. If not stable → INVALID, discard.
Read path: Strategy Tester DOM scrape of the Performance overview strip.

### Run 1 — post-compile live deep result

| Read | Net % | Net USDT | MaxDD % | PF | Win % | Trades | DEEP | Range |
|---|---|---|---|---|---|---|---|---|
| 1 | +623.91 | +6,239.07 | 10.86 | 2.265 | 43.73 (223/510) | 510 | yes | 2020-01-01 → 2026-06-14 |
| 2 | +623.91 | +6,239.07 | 10.86 | 2.265 | 43.73 (223/510) | 510 | yes | 2020-01-01 → 2026-06-14 |

→ 2 consecutive agreeing reads. **STABLE.**

### Run 2 — independent recompute (timeframe round-trip 15→60→15, no parameter change)

| Read | Net % | Net USDT | MaxDD % | PF | Win % | Trades | DEEP |
|---|---|---|---|---|---|---|---|
| 1 | +623.91 | +6,239.07 | 10.86 | 2.265 | 43.73 (223/510) | 510 | yes |
| 2 | +623.91 | +6,239.07 | 10.86 | 2.265 | 43.73 (223/510) | 510 | yes (CAGR 35.89%) |

→ 2 consecutive agreeing reads. **STABLE.**

### Agreement

Run 1 ≡ Run 2 to the digit. No discrepant fields. **Extra (3rd) run not required.**

## Verified headline (commission-only, 0.05%/side)

| Metric | Value |
|---|---|
| Net profit | **+623.91%** (+6,239.07 USDT on 1000) |
| Max drawdown | **10.86%** |
| Profit factor | **2.265** |
| Win rate | **43.73%** (223 / 510) |
| Total trades | **510** |
| CAGR | 35.89% |
| Avg bars in trade | 45 |

Matches documented baseline (+624.32% / 10.85% / 2.265 / 510); the small net-% delta is two
extra days of bars to 2026-06-14.

## Cost caveat (must travel with this number)

This is a **commission-only, in-sample BTC, both-sides** figure. It includes **no** slippage and
**no** perpetual funding. It is the headline backtest, **not** an expected live return. Slippage
and funding (estimated ~30% bite in the prior long-only audit) and the documented cross-asset
overfitting must be applied before any live expectation is set. See `../limitations.md`.

## Verdict

**PASS.** Headline BTC Deep Backtest reproduced cleanly; two independent, poll-to-stable runs
agree exactly. Safe to proceed to building the public tables/charts from this frozen candidate.
