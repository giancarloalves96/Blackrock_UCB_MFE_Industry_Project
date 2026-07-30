# PM-experiment result series for the slide build

Chart-ready CSVs for pm-3arm-haiku and pm-3arm-sonnet (stage 1). Generated from
the committed runs in `reports/hkpm/` — regenerate with
`PYTHONPATH=. uv run python scripts/export_slide_results.py` (script committed
alongside). Windows: **haiku = 2016-01→2026-06 (n≈126)**, **sonnet =
2021-07→2026-06 (n≈60)**. Cross-model comparisons should be drawn on the shared
window only.

## Files

### pm_arms_monthly.csv — the time series (long format)
One row per (date, model, pod, arm, measure). Columns:
- `monthly` — that month's result, in `unit`:
  - `excess_return` (equities): strategy simple return in excess of the 1-month
    bill; position formed at month-end t-1 earns month t.
  - `pp_yield` (rates pods): trade P&L as Σ leg-weight × Δyield, in percentage
    points. **Not a return** — sign is opposite a price P&L. Months with no
    emitted trade are 0 (flat book), not missing.
  - `scaled_return` (composite): see below.
- `cumulative` — compounded for returns, summed for pp_yield.
- `rolling_sharpe_12m` — 12-month rolling mean/std, ×√12. Blank for the first
  11 months. Plot with the full-sample value as a reference line.

Arms: `full` (reports + attribution), `conv` (convictions only), `raw` (no
analysts, raw measurements), `mech` ($0 arithmetic; model-independent, so
identical rows serve both models). Equities extras: `board_mean` (no PM),
`ridge` (walk-forward fitted null), `buy_hold`.

Measures: `mapped` = the preregistered polarity-map of driver convictions
(primary); `pm_trade` = the PM's own sized SPY position (secondary);
`yield_pnl` = rates trade P&L (secondary).

### pm_arms_summary.csv — one row per series
n, mean_monthly, ann_sharpe (√12), t_stat, hit_rate (months with a nonzero
position only), max_dd.

### equities_beta_hedged_monthly.csv / _summary.csv
Each equities series regressed on SPTR excess returns (full-sample OLS beta —
in-sample by construction, disclosed): `hedged = strat − β·market`.
`alpha_monthly` is the hedged mean, `alpha_t` its t-stat. Answers "is the
performance beta or timing?"

### composite_5pod (inside pm_arms_monthly/summary)
An ILLUSTRATIVE post-hoc "final portfolio": no fund layer exists yet, so this is
each pod's monthly series scaled to 10% annualized vol (full-sample σ —
in-sample, disclosed) and equal-weighted across the 5 pods, per arm. It mixes
return-space (equities) and yield-space (rates) series after vol-scaling; treat
it as a shape, not a P&L. NOT preregistered; descriptive only.

## Key numbers (for the observations)

Equities mapped Sharpe (haiku full-window / sonnet 5y):
full −0.51/−0.64 · conv −0.56/−0.66 · raw −0.44/−0.39 · mech=board_mean
−0.45/−0.64 · buy_hold +0.86/+0.61.

Equities beta-hedged alpha t-stats — mapped arms: haiku −0.45..+0.26, sonnet
−0.86..−0.29 (all ≈ 0); PM-sized trades: haiku +0.74..+1.22 (all positive),
sonnet −0.21..+0.30. Every book carries β ≈ −0.16..−0.20.

Composite Sharpe per arm: haiku mech 0.34 > conv 0.23 > full 0.00 > raw −0.25;
sonnet full 0.65 > mech 0.38 > raw 0.06 > conv −0.17 (t=1.46 on sonnet full —
suggestive, not significant, n=60).

## Observations — the two designed experiments, per model

### A. Attention effects: full (PM reads the 11 analysts) vs raw (PM fetches
### all measurements into one context)
- **Haiku:** full ≥ raw on 4/4 rates pods (largest gap front_end −0.03 vs
  −0.23 d_ic); composite +0.25 Sharpe (0.00 vs −0.25, p_boot 0.57). Equities
  the exception: raw −0.44 vs full −0.51 (p 0.86).
- **Sonnet:** full ≥ raw on 3/4 rates pods; composite +0.59 Sharpe (0.65 vs
  0.06, p_boot 0.19). Equities again the exception (raw −0.39 vs full −0.64).
- **Reading:** directionally consistent support for the partitioned
  architecture wherever the analysts have signal, gap WIDENING with model
  quality (+0.25 → +0.59); it inverts exactly on the panel whose analysts are
  anti-predictive (equities) — reading raw data lets the PM partially escape
  bad analysts. Never significant.

### B. Reasoning impact: full (reports + attribution) vs conv (signed
### conviction only)
- **Haiku:** the prose flips the position sign in 11-32% of months but adds
  nothing — conv ≥ full on 4/4 rates pods, composite BETTER without reasoning
  (0.23 vs 0.00), and conv is the one arm significantly below the no-PM
  baseline on equities (p .042). Zero-to-negative reasoning impact.
- **Sonnet:** prose flips far fewer decisions (sign agreement 81-98%) but
  pays where it does: full ≥ conv on 3/4 rates pods; composite +0.82 Sharpe
  (0.65 vs −0.17, **p_boot 0.10 — the largest contrast in the study**).
  Equities indistinguishable.
- **Reading:** reasoning impact is a property of the READER: the weak PM is
  moved often and randomly by the prose; the strong PM overrides rarely and
  profitably. The p=0.10 is stage-2's headline hypothesis, not a finding.

### Supporting observations
1. **vs the mechanical arm (composite):** Haiku — mech 0.34 beats every LLM
   arm; Sonnet — full 0.65 vs mech 0.38 (t 1.46, n.s.). Destroying → neutral.
2. **The equity Sharpes are beta, not timing:** every equity book carries
   β ≈ −0.17; hedged alphas of the mapped views ≈ 0. The tilt, not the calls,
   is the cost.
3. **Sizing survives the beta hedge (Haiku):** PM-sized SPY books show
   positive hedged alpha in all three LLM arms (t ≤ 1.2) while the views sit
   at zero.
4. **Windows differ** (126 vs 60 months); the composite is illustrative
   post-hoc — label both on any chart that mixes them.
