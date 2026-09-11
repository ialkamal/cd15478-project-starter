# Assumptions and Sources

Companion to `nimbus_decision_starter.ipynb` and `decision_memo.md`.
Nimbus Streaming — Standard tier pricing decision ($12.99 → $14.99, +15.4%).
All churn *lifts* are carried in **percentage points (pp)**; rates are fractions. The single conversion point is `annual_lift_from_3mo`.

---

## 1. Inputs and data lineage

| Input | Source file | Field(s) used | Value used | Notes |
|---|---|---|---|---|
| Current price | `finance_forecast.csv` | `standard_price_today_usd` | $12.99 | Constant across all 12 forecast months (asserted in the notebook) |
| Proposed price | `finance_forecast.csv` | `standard_price_proposed_usd` | $14.99 | +$2.00, +15.4% |
| Contribution margin | `finance_forecast.csv` | `contribution_margin_pct` | 62% | Mean of the 12 monthly values |
| CAC | `finance_forecast.csv` | `cac_per_sub_usd` | $35 | Mean of the 12 monthly values |
| Churner remaining tenure | `finance_forecast.csv` | `avg_remaining_tenure_months` | 30 months | Base-wide average; see limitation L1 |
| Base size | `finance_forecast.csv` | `beginning_subscribers` | 4,000,000 | First forecast month |
| New sign-ups, 12 months | `finance_forecast.csv` | `expected_new_subscribers` | 1,080,000 | Sum over 12 months |
| Organic 3-month churn | `finance_forecast.csv` | `expected_organic_churn / beginning_subscribers` | 6.46 pp | Monthly 2.2%, compounded over 3 months |
| Pilot outcomes and covariates | `pilot_data.csv` | 40,000 subscribers; 13,320 pilot / 26,680 control | — | `churned_3mo`, `urban`, `income_bracket`, `tenure_months`, `engagement_score`, `plan_type` |
| Prior evidence | `industry_pricing_history.csv` | `pct_price_increase`, `observed_3mo_churn_lift_pp` | 20 of 42 events | Filtered to 10–20% increases |
| Stated intent | `pre_announcement_survey.csv` | `stated_intent_at_14_99` | 1,500 respondents, 21.5% "Cancel" | |
| Segment WTP | `wtp_segments.csv` | shares, stated cancel rates, WTP percentiles | 4 segments | Used for the hybrid extension and as a sanity check |

No financial constant is hard-coded in the notebook; every one is read from `finance_forecast.csv`.

**Sanity check that passed.** The base-weighted stated cancel rate implied by `wtp_segments.csv` is 19.3%, against 21.5% measured directly in the survey. Two independently supplied files agree to about 2 pp, which is reassuring on both.

---

## 2. Causal correction (Section 2)

**The problem.** Pilot assignment was not random. Pilot markets are 71% urban versus 47% in control (SMD +0.51) and about half a pooled SD higher on income (SMD +0.47). `tenure_months` (−0.01), `engagement_score` (+0.06) and `plan_type` mix are balanced. Because urban and higher-income subscribers churn more at baseline, the naive difference attributes their higher baseline churn to the price change.

**The correction.** Inverse probability weighting. Propensity model: logistic regression of `in_pilot` on the four numeric covariates plus drop-first `plan_type` dummies. Estimator: **Hájek** (weight-normalised) IPW — treated weighted by 1/e(x), controls by 1/(1−e(x)), then the difference of the two weighted means of `churned_3mo`.

**Choices made.**
- *Hájek over Horvitz–Thompson*: normalising by the sum of weights is lower-variance and keeps the estimate inside [0,1] in finite samples. Difference here is negligible given the good overlap.
- *No trimming*: propensity scores span 0.07–0.70 in both arms, 100% of the sample is on common support, and the largest single weight is ≈13. Positivity holds, so trimming would discard information for no bias reduction.
- *Bootstrap*: 500 resamples, **refitting the propensity model inside each resample**, so the CI reflects uncertainty in the propensity model itself, not just the weighting step.

**Result.** Naive 4.86 pp → IPW **3.70 pp** (95% CI [3.12, 4.29], SE 0.30). Bias correction −1.16 pp, ≈24% of the naive estimate, in the direction Experimentation predicted.

**Robustness (AIPW).** IPW is unbiased only if the propensity model is right. We added an outcome model — separate logistic regressions of `churned_3mo` within each arm on the same covariates — and combined the two via the augmented IPW estimator, which is consistent if *either* model is correct. AIPW gives **3.70 pp**, within 0.01 pp of plain IPW with an effectively identical bootstrap CI. The conclusion does not depend on the propensity specification. IPW remains the headline estimate; had the two diverged materially, we would have carried the wider of the two into the Bayesian update.

**What IPW cannot fix.** Unobserved confounding. If pilot markets differ on something not in the dataset that also drives churn — local competitor launches, regional content licensing gaps, service quality — the correction is incomplete. IPW only balances what we measured.

---

## 3. Bayesian updating (Section 3)

**Prior.** Industry events with price increases of 10–20% (n = 20), bracketing Nimbus's +15.4%. Empirical mean **3.26 pp**, SD **0.93 pp**. The window is a judgement call: too narrow and the prior is noise, too wide and it includes 5% increases that say little about a 15% one. Widening to 8–22% (n = 30) moves the prior mean by about 0.2 pp, to 3.03; the posterior is dominated by the pilot either way.

**Model.** Conjugate Normal-Normal with known variance. Precisions add: `post_var = 1/(1/prior_sd² + 1/lik_sd²)`. Normal is a reasonable shape for a near-symmetric, bounded-away-from-0-and-1 effect, and it keeps the update transparent. The cost is that the Normal has unbounded support — negative lifts have non-zero density — which is immaterial here since the posterior sits 15 SDs above zero.

**Update 1 — pilot.** Prior (3.26, 0.93) + IPW (3.70, 0.30) → **Posterior 1 = 3.66 pp, SD 0.29**. The pilot carries 91% of the weight: 40,000 subscribers are far more informative about Nimbus than 20 events at other companies.

**Update 2 — survey.** Stated cancel 21.5% → ×0.50 stated-to-revealed = 10.8% expected revealed cancellation → minus 6.46 pp organic 3-month churn = **4.31 pp**. SE = binomial SE of the stated proportion (1.06 pp) × 0.50 = **0.53 pp**. Posterior 1 + survey → **POSTERIOR_MU = 3.81 pp, POSTERIOR_SD = 0.25 pp** (90% credible interval 3.39–4.22).

**Known weakness of the survey likelihood.** The 0.50 ratio is a literature-informed judgement, not an estimate from these data, and its uncertainty is *not* in the 0.53 pp SE — which therefore overstates how much we should trust the survey. Sensitivity: a ratio of 0.35 puts the survey lift at 1.1 pp and pulls the posterior to ≈3.5 pp; a ratio of 0.65 puts it at 7.5 pp and pushes the posterior to ≈4.1 pp. Both remain well below the 5.1 pp break-even, so the recommendation is unchanged across the plausible range. Note also that survey respondents span all tiers (881 of 1,500 are Standard); we used the pooled rate, and the Standard-only rate (20.8% versus 21.5%) is materially similar.

---

## 4. Cost-benefit model (Section 4)

| # | Assumption | Rationale | If wrong |
|---|---|---|---|
| A1 | The 3-month lift **is** the annual incremental churn — no compounding | Price-shock churn is a one-off reaction in the quarter after the announcement; a subscriber can only quit over the same price increase once | Compounding quarterly (`1−(1−l)⁴`) would roughly quadruple the churn cost and turn Full negative. This is the most consequential structural choice in the model, and it is standard practice for price-shock analysis |
| A2 | Incremental churners do **not** pay the new price | They leave in the quarter; uplift revenue accrues to the retained base only | Counting them on both sides would overstate Full by ≈$2.3M |
| A3 | Cost per base churner = remaining lifetime margin (at the **old** price × 30 months × margin) + one replacement CAC | We lose the revenue we actually had, not the price they refused; Finance's plan holds the base flat, so a lost subscriber must be bought back | See L1 — this is the dominant sensitivity |
| A4 | `NewOnly`: ~6 months average in-window billing; CAC is a **saving**, not a charge; same churn lift applied | Sign-ups arrive through the year, so average exposure is half the window. A prospect who balks at $14.99 is a non-acquisition — Nimbus never pays their CAC | Charging replacement CAC instead of crediting it would move `NewOnly` to ≈−$3.6M. Going the other way and halving the lift for new joiners (they never anchored on $12.99) lifts it to ≈+$3.6M. Even the most generous variant is a quarter of Full |
| A5 | `Delay`: 6 months of uplift inside the window, but the **full** churn cost | Postponing defers the revenue, not the damage: whenever the price moves, the lifetime value of everyone it drives away is lost, and lifetime loss is not clipped at the window boundary. Plus $250k of incremental research spend | If you instead halve the churn cost (assuming part of the exit wave falls outside the window), `Delay` scores ≈+$7.3M — about $21M better, still below Full's $15.1M. The ranking is unchanged |
| A6 | No discounting; downgrades to Basic treated as churn; no competitive response | At ~8% a year, discounting would shave ≈4% off the 12-month uplift but ≈10% off the 30-month lifetime-loss term, i.e. it would *improve* Full by roughly $1M — omitting it is conservative. Treating downgrades as full churn is likewise conservative | A competitor undercutting on price in the window is not modelled — it is a tail risk that the monitoring tripwire, not the model, is meant to catch |

**Monte Carlo.** 10,000 draws (seeded, `default_rng(7)`), four uncertain inputs drawn independently: lift ~ Normal(3.81, 0.25); margin ~ Normal(0.62, 0.02); CAC ~ Normal(35, 3); LTV months ~ Normal(30, 3). Each spread is set so ±2 SD spans the range Finance provided — i.e. their band is read as an approximate 95% range, not a hard bound. **Independence is an assumption and a limitation**: margin and CAC plausibly co-move with the macro conditions that also drive churn, and correlated draws would widen the tails (though not shift the mean).

---

## 5. Decision theory (Section 7)

| Rule | Parameters and rationale | Choice |
|---|---|---|
| Expected value | Mean of 10,000 simulated profits | Full ($15.1M) |
| CRRA certainty equivalent | γ = 2 (moderate risk aversion, standard corporate default); wealth baseline $500M ≈ Nimbus's annual contribution profit, which keeps utility well-defined and sets the scale at which a loss actually hurts | Full ($15.0M CE) |
| Minimax regret | Per-draw regret = best profit that draw − this option's profit; choose the smallest maximum regret | Full (max regret $6.7M, versus $33.8M for Hold and $34.6M for Delay) |

`RECOMMENDED = "Full"`, matching the memo. The three rules agree because Full's downside is thin: it is positive in 99.8% of draws and its SD (≈$5M) is small next to a $500M baseline, so the risk adjustment is under $0.1M. Note that γ and the wealth baseline are the least data-grounded parameters in the analysis; they happen not to matter here, and we would flag them loudly if they did.

**Robustness.** Full is the best of the four options in **99.8%** of scenarios. Break-even lift **5.11 pp** (posterior probability of exceeding: ~1.4×10⁻⁷). Break-even churner tenure **42.3 months** (forecast 30). Break-even margin **16.2%** (forecast 62%).

---

## 6. Segment extension and the hybrid option (Section 9)

Segment-level lifts are constructed by scaling `POSTERIOR_MU` by each segment's stated cancel rate relative to the base-weighted average (multipliers 1.76 / 0.93 / 0.31 / 0.52), so segment lifts aggregate back to the posterior mean exactly and the four segment profits sum to the directly computed `Full` profit ($15.12M) — a useful internal consistency check. `Hybrid` holds the Price-sensitive segment at $12.99 and raises the rest: EV **$20.4M**, ≈$5.3M better than Full, and it wins on all three decision rules among the five options.

**Why it is not the recommendation.** (i) Segment lifts are *assumed proportional to stated intent*, not measured — the pilot was not stratified by WTP segment, so there is no revealed-preference evidence that price-sensitive subscribers churn 1.8× the average. (ii) Segment membership is a survey construct with no production assignment rule; implementing it requires an observable proxy, and every misassignment leaks revenue. (iii) Visible price discrimination against loyal, long-tenured subscribers carries reputational risk that is not in the $5M. The memo therefore proposes measuring it (2% holdout, ~$0.3M) rather than banking it.

---

## 7. Known limitations

- **L1 — Churner lifetime value is the weakest link.** The model applies the base-wide 30-month average remaining tenure to price-induced churners, who are by construction the most price-sensitive subscribers and plausibly shorter-tenured than average. If anything this makes the $15.1M *conservative*; the recommendation only fails if incremental churners would have stayed 42+ months. This single parameter drives a $14.7M swing — more than the other three drivers combined — and it is an assumption, not a measurement.
- **L2 — Unobserved confounding in the pilot.** IPW and AIPW balance observed covariates only (L1 above notwithstanding, this is the main threat to the causal estimate).
- **L3 — Stated-to-revealed ratio.** A judgement parameter whose uncertainty is not propagated; sensitivity is documented in §3.
- **L4 — Static competitive and content environment.** No competitor price response, no content-slate effects, no cross-tier substitution beyond treating downgrades as churn.
- **L5 — Independent Monte Carlo draws.** Correlated inputs would fatten the tails.
- **L6 — Single-horizon view.** Twelve months only. A price increase changes the long-run price-expectation anchor and the composition of the base in ways this analysis does not capture.

## 8. Reproducibility

`nimbus_decision_starter.ipynb` runs top-to-bottom from a fresh kernel and is deterministic: `np.random.default_rng(7)` seeds the bootstrap (500 resamples), the AIPW bootstrap (300 resamples) and the 10,000-draw Monte Carlo, in that order. Data are read from `../data/` with a fallback search of common locations. Libraries: pandas, numpy, statsmodels, scipy, seaborn, matplotlib.
