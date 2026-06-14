# Audit log — Cost / slippage / funding sensitivity

**Date (UTC):** 2026-06-14
**Candidate (frozen):** `risk_based_baseline_slim.txt` — Engulfing_TURBO (RiskBased, slim), both-sides.
**What changed between runs:** only `commission_value` (a cost assumption). No strategy logic or
parameters touched. Editor restored to commission 0.05 afterward and re-verified to the digit.

## Method

- TradingView supports **percent** commission (`strategy.commission.percent`). Slippage is
  modeled as **additional percent commission per side**, NOT the fixed-tick slippage field — a
  fixed tick cannot hold a constant % across BTC's ~$4k–$110k range over 2020–2026.
- Each scenario: edit `commission_value` → compile → poll the Strategy Tester DOM until trade
  count, net %, MaxDD %, PF, and win rate are identical across ≥2 consecutive reads → record.
- Funding cannot be modeled in TradingView → estimated analytically (see below), labeled
  **ESTIMATED**.

## Measured scenarios (poll-to-stable)

### Scenario A — commission only (0.05%/side, RT 0.10%)

| Read | Net % | MaxDD % | PF | Win | Trades | DEEP |
|---|---|---|---|---|---|---|
| (baseline, prior run) | +623.91 | 10.86 | 2.265 | 223/510 | 510 | yes |

= the verified reproduction baseline (commission was always modeled).

### Scenario B — fee + slippage (0.07%/side, RT 0.14%)

| Read | Net % | Net USDT | MaxDD % | PF | Win | Trades | CAGR | DEEP |
|---|---|---|---|---|---|---|---|---|
| 1 | +518.40 | 5,184.00 | 11.39 | 2.091 | 215/510 | 510 | 32.61% | yes |
| 2 | +518.40 | 5,184.00 | 11.39 | 2.091 | 215/510 | 510 | 32.61% | yes |

→ 2 consecutive agreeing reads. **STABLE.**

### Scenario C — stress (0.10%/side, RT 0.20%)

| Read | Net % | Net USDT | MaxDD % | PF | Win | Trades | CAGR | DEEP |
|---|---|---|---|---|---|---|---|---|
| 1 | +388.14 | 3,881.40 | 12.33 | 1.865 | 207/510 | 510 | 27.84% | yes |
| 2 | +388.14 | 3,881.40 | 12.33 | 1.865 | 207/510 | 510 | 27.84% | yes |

→ 2 consecutive agreeing reads. **STABLE.**

### Restore — commission 0.05 (frozen baseline)

| Read | Net % | MaxDD % | PF | Win |
|---|---|---|---|---|
| 1 | +623.91 | 10.86 | 2.265 | 223/510 |
| 2 | +623.91 | 10.86 | 2.265 | 223/510 |

→ matches Scenario A to the digit. Cost runs were clean and fully reversible; **editor is frozen.**

Trade count is constant at **510** across A/B/C — cost changes do not alter entries, only
which marginal trades end green (winners 223 → 215 → 207 as cost rises).

## Measured cost sensitivity (compounding-inclusive)

| Step | Δ round-trip cost | Δ net | net-pts per 0.01% RT |
|---|---|---|---|
| A→B | +0.04% | −105.51 | −26.4 |
| B→C | +0.06% | −130.26 | −21.7 |
| A→C | +0.10% | −235.77 | −23.6 |

**Central ≈ −24 net-pts per 0.01% round-trip.** Non-linear (more sensitive at lower cost base).
**Slippage alone (A→B) removes 16.9% of the commission-only net.** This both-sides config (510
trades) is more cost-sensitive than the earlier long-only probe (−14) due to higher turnover.

## Funding — ESTIMATED (not measurable in TradingView)

- Binance BTC perp funding mean ≈ +0.01%/8h. Tested 0.01 / 0.02 / 0.03%/8h.
- Treated as a **cost on held notional regardless of side** (conservative: a both-sides trend
  book tends to pay on both legs).
- Hold time: **count-weighted 11.25h** (avg 45 bars) and **notional-weighted ~24h** (the runner
  leg = ~95% of notional, held 1–3 days, so funding $ is dominated by runners — the conservative,
  more realistic anchor).
- Funding (% notional over hold) → net-% impact via the measured −24 pts/0.01%-RT sensitivity.

**All-in net (Scenario B + estimated funding):**

| Funding/8h | count-weighted | notional-weighted |
|---|---|---|
| 0.01% | +484.7% (CAGR 31.5%) | +446.4% (CAGR 30.1%) |
| 0.02% | +450.9% (CAGR 30.3%) | **+374.4% (CAGR 27.3%)** ← central |
| 0.03% | +417.1% (CAGR 29.0%) | +302.4% (CAGR 24.1%) |

## Cost-bite summary (vs commission-only +623.91%)

- Slippage only (B): **−16.9%**.
- Realistic all-in (central, B + funding 0.02 notional-weighted): **−40%**.
- Range across funding assumptions: **−22% to −52%**.

## Verdict

Realistic costs take a **~40% central bite** out of the in-sample paper return. The edge
**survives**: even the conservative all-in holds PF > 1.7 and CAGR > 24%/yr. But the public
headline must NOT be the +624% commission-only figure — live expectation belongs ~40% below it.
MaxDD is barely affected by cost (10.86% → 12.33% at the C extreme; funding is a smooth drag, not
a new dip source). Funding layer numbers are ESTIMATES; A/B/C are measured.
