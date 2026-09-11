# Decision Memo: Standard Tier Pricing

**To:** CFO, Nimbus Streaming
**From:** Decision Science, Strategy
**Date:** 11 September 2026
**Re:** Recommendation on the proposed $12.99 → $14.99 Standard tier price change

---

## Recommendation

**Proceed with the full rollout.** Expected 12-month incremental contribution: **$15.1M** (90% interval **$6.9M to $22.9M**). Full rollout is the highest-value choice in **99.8%** of 10,000 simulated scenarios, and it wins under all three decision rules we applied — expected value, risk-adjusted certainty equivalent, and minimax regret. The alternatives cost money: *New customers only* is roughly break-even (−$0.8M), *Delay* is −$13.8M, and *Hold* is $0 by definition.

## What's behind the number

**1. The pilot overstated the damage; we corrected it.** Pilot markets were 71% urban versus 47% in control and sat half a standard deviation higher on income — both groups that churn more anyway. The raw pilot read a **4.9 pp** lift in 3-month churn. Reweighting subscribers by their inverse probability of pilot assignment (IPW) brings that down to **3.7 pp** (95% CI 3.1–4.3). The correction is worth **1.2 pp**, about a quarter of the headline. A doubly-robust (AIPW) estimator lands on the same number, so this does not depend on getting the propensity model exactly right.

**2. Three sources of evidence agree on ~3.8 pp.** We started from 20 comparable industry price increases of 10–20% (mean 3.3 pp, SD 0.9), updated with the corrected pilot, then with the 1,500-person intent survey (21.5% said "cancel", halved to 10.8% expected actual, less 6.5 pp of organic churn = 4.3 pp). The final posterior is **3.8 pp ± 0.25 pp** — roughly **152,000 incremental cancellations** over the quarter.

**3. How that becomes $15.1M.** The 3.85M subscribers who stay pay $2.00 more for 12 months: **+$57.2M** of contribution at 62% margin. Against it we charge the full remaining lifetime margin of the 152,000 we lose (30 months at today's price: **−$36.8M**) and the cost of reacquiring them at $35 CAC (**−$5.3M**). Net: **+$15.1M**. Churners are excluded from the revenue side, so no subscriber is counted as both a loss and a payer.

## What could change the answer

**Break-even is a 5.1 pp churn lift** — 5.1 standard deviations above our posterior mean of 3.8 pp, and above even the *uncorrected* pilot estimate of 4.9 pp. The posterior probability of landing there is under 1 in a million. The decision does not turn on the churn estimate.

| Driver (flexed) | Profit swing |
|---|---|
| Tenure of an incremental churner (30 ± 6 months) | **$7.8M – $22.5M** (±$7.4M) |
| Churn lift (± 1 posterior SD, 3.6–4.1 pp) | $12.2M – $18.1M (±$2.9M) |
| CAC ($35 ± $10) | $13.6M – $16.6M (±$1.5M) |
| Contribution margin (62% ± 4 pp) | $13.8M – $16.4M (±$1.3M) |

**The recommendation flips only on lifetime value, not on churn.** The single assumption doing the most work is that an incremental churner would otherwise have stayed **30 more months** — Finance's base-wide average. If price-sensitive leavers would in fact have stayed **42 months or longer**, full rollout stops paying for itself. That is the number to challenge. Margin would have to collapse to 16% to change the call, so it is not a live risk.

**One material upside we are not yet claiming.** Segment analysis suggests the price-sensitive quartile (28% of the base, median willingness-to-pay $12.50 — below the proposed price) is the only segment that *loses* money under a blanket increase. Holding them at $12.99 would add roughly **$5M**. We are not recommending that today because segment churn rates are inferred from stated intent, not measured — see step 3.

## Recommended next steps

1. **Approve the increase for the next billing cycle**, and pre-authorise a capped **$3M retention budget** that releases only if the tripwire below trips. A price increase is hard to reverse; a targeted win-back offer is the practical lever.
2. **Monitoring tripwire — weekly incremental cancellations against a matched baseline.** Break-even pace is 204,000 incremental cancellations for the quarter; we forecast 152,000. **If cumulative incremental cancellations exceed 100,000 by day 30 or 150,000 by day 60, pause remaining announcement waves, release the retention budget, and re-run this model.** Secondary tripwire: Basic-tier downgrades running more than 25% above forecast, which would signal we are trading margin rather than losing subscribers.
3. **Hold out 2% of the base at $12.99 for one quarter** to measure the churn lift by willingness-to-pay segment with revealed behaviour rather than survey intent. Cost: about **$0.3M** in forgone uplift. It is the cheapest way to test the ~$5M carve-out, and it converts the largest remaining assumption — churner lifetime value — into a measurement.

*See `nimbus_decision_starter.ipynb` for full methodology, data lineage, and uncertainty quantification, and `assumptions_and_sources.md` for inputs and known limitations.*
