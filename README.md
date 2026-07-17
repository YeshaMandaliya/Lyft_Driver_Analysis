# Early Driver Disengagement: Designing a Risk-Based Retention Experiment

*A case study on the 2016 Lyft (San Francisco) driver dataset - framed and executed as a business decision, not just a modeling exercise.*

---

## TL;DR

By a driver's **14th day**, two simple signals — **how many days they were active in their first two weeks** and **how long they've been idle** — flag a stable group of drivers who account for most future "going quiet." A simple two-signal rule matches a complex 11-variable model, so the right answer is **a simple, explainable rule plus a controlled test — not a black box.**

The deliverable is a **test (pilot) design** plus a **capacity view** for spending limited outreach — not a churn score, and not a made-up ROI number.

> **Headline:** Drivers active on **4–7 of their first 14 days** are ~⅓ of new drivers but ~½ of everyone who later goes quiet — an efficient group to test retention outreach on.

---

## The question

Reaching out to new drivers costs time and money, so Lyft can't treat everyone the same. The question:

> **Can a driver's first two weeks of activity flag who is likely to go quiet — early enough, and clearly enough — to focus retention attention, and what should Lyft do about it?**

This is framed as a **focus-and-learn decision**, not a prediction contest. The distinction is the whole point: risk tells you *who might drift away*; only a controlled test tells you *who Lyft should actually prioritize*.

---

## Why this framing (not "driver lifetime value")

This is the well-known Lyft driver dataset, usually used for a lifetime-value analysis. This project reframes it as an **early-retention / marketplace-supply** problem and — importantly — stays strict about what observational data can and can't prove. The payoff is a defensible experiment design, which reads as more operationally mature than a churn dashboard.

---

## Data & audit (trust earned, not assumed)

| File | What it is | Rows |
|---|---|---|
| `driver_ids.csv` | one row per driver (onboarding date) | 937 |
| `ride_ids.csv` | one row per completed ride (distance, duration, prime-time) | 193,502 |
| `ride_timestamps.csv` | one row per ride step (requested → dropped off) | 970,405 |

- **The biggest weakness is the joins, not missing values.** Only 854 of 937 drivers appear in both the onboarding and ride files (a matching 83/83 gap, plus ~4.5% slippage at the ride level). A careless merge would silently drop ~100 drivers and never mention it.
- **Trusted group: 837 drivers** with complete, linkable histories. The 83 onboarding-only IDs are **not** labeled "never drove" — their cause is unresolved, so they're set aside, not mislabeled.
- **`ride_duration` checks out exactly** against the pickup→dropoff times, so estimated ride value sits on a verified input.
- **Clock: UTC.** Reading the dates as San Francisco local time creates 177 impossible "rides before onboarding," so UTC is the consistent choice (local time kept only as a sensitivity check).
- **Selection effect documented, not hidden:** completeness drops across later onboarding groups and excluded drivers tend to be lower-volume — so the real risk is likely *higher* than modeled, and all claims are scoped to fully-linkable drivers, not all 937.

---

## Approach — a disciplined route from a fuzzy question to a defensible decision

1. **Frame the decision** — owned by Driver Ops; explicit "can say / can't say" list.
2. **Structure** — decision gates (trust the data → is it material → is there an early signal → does it beat a one-line rule → is risk concentrated → is it stable).
3. **Establish honest data** — the audit above; a clean, leak-free outcome definition.
4. **Analyze** — inactivity curves for timing, a simple rule as the benchmark, a complex model as the challenger.
5. **Find the "so what"** — a stratified pilot design and a capacity view.
6. **Communicate** — this README and the [executive memo](EXECUTIVE_MEMO.md).

**Outcome:** a driver's first stretch of **21 straight days with no rides** after day 14. Deliberately **not** called churn — about a third of these drivers return within three weeks. Thresholds of 14/21/28 days are treated as *different business outcomes* (a break vs a real stop), not interchangeable settings.

**No-leakage rule:** every signal uses only what's known *before* day 14; the current idle streak at day 14 is fair game; the "event" is the moment the 21-day streak completes, never worked out backward from a driver's last-ever ride.

---

## Key results

**Slow starters go quiet sooner and more often.**

![Inactivity curves by early-activity group](images/inactivity_curves.png)

**Early activity is a strong, simple risk flag — near-term inactivity falls from 34% to 2%.**

![Near-term inactivity by early-activity group](images/risk_by_segment.png)

| First-14-day activity | Drivers | Share of "goes quiet" events | Chance of going quiet within 28 days |
|---|---|---|---|
| 1–3 active days | 100 | 21% | 34.0% |
| **4–7 active days** | **269** | **49%** | **26.0%** |
| 8–10 active days | 209 | 19% | 13.9% |
| 11–14 active days | 256 | 12% | 2.3% |

**The flag is dependable — it separates risk in every weekly onboarding group.**

![Risk separation across onboarding groups](images/cohort_stability.png)

**A simple rule matches a complex model — so keep it simple.**

![Model comparison](images/model_comparison.png)

| Model | Accuracy score* | Verdict |
|---|---|---|
| Active-day rule (1 signal) | 0.712 | Strong on its own |
| Idle-streak rule (1 signal) | 0.708 | Strong on its own |
| Two-signal model | 0.732 | Best practical choice |
| 11-variable model | 0.735 | **Rejected** — +0.003 only, and unstable |

*\*0.50 = a coin flip; ~0.71–0.73 means the rule ranks who's more at risk correctly ~71–73% of the time. Useful for focus, not a precise per-driver prediction.*

**Focusing on the riskiest 30% of drivers covers about half of all inactivity.**

![Capacity curve](images/capacity_curve.png)

| If Lyft can reach... | ...it covers this share of drivers who go quiet |
|---|---|
| Top 10% | 18.7% |
| Top 20% | 34.9% |
| Top 30% | 51.4% |
| Top 40% | 64.1% |
| Top 50% | 75.4% |

---

## Recommendation

Run a **controlled test** (outreach vs. do-nothing-different), split at day 14:

- **Start with the 4–7-active-day group** — common enough to measure a real effect, and engaged enough to be worth keeping.
- **Measure:** the difference in how many go quiet within 28 days, outreach vs. control.
- **Sort by current idle streak** (under 3 / 3–7 / 7–14 days).
- **Include smaller test groups from other activity levels** — otherwise you can't tell whether *focusing* beats *contacting everyone*.

**Interim (low-cost only):** cheap, reversible outreach can start with the 4–7-day drivers already idle 3+ days by day 14 (~39% go quiet) — as a temporary rule of thumb, not a reason to spend on incentives.

---

## What this project does *not* claim

This is observational, single-group, 2016 data. It supports *spotting* risk only. It does **not** show that outreach prevents inactivity, saves rides, or pays off; that high-risk drivers are the most *reachable*; that focusing beats contacting everyone; that incentives are justified; or that any of this applies to Lyft today. "Going quiet" is a re-engagement signal, not confirmed churn (~35% come back within three weeks).

---

## Reproduce

```bash
git clone <this-repo>
cd <this-repo>
pip install -r requirements.txt
# open the notebook and Run All (top to bottom)
```

The notebook uses UTC as the primary clock; timezone, data-anomaly, and threshold-sensitivity checks are in a clearly labeled appendix. A random seed is set for reproducibility.

```
├── data/                          # raw challenge CSVs (or a source note)
├── images/                        # charts used in this README + the memo
│   ├── inactivity_curves.png
│   ├── risk_by_segment.png
│   ├── cohort_stability.png
│   ├── model_comparison.png
│   └── capacity_curve.png
├── notebooks/
│   └── driver_disengagement.ipynb
├── EXECUTIVE_MEMO.md
├── requirements.txt
└── README.md
```

## Methods & tools

Python · pandas · NumPy · lifelines (Kaplan–Meier, Cox proportional hazards) · matplotlib. Survival analysis with right-censoring; leave-one-cohort-out validation; two-proportion power calculations for pilot sizing.

---

*Data: 2016 Lyft Driver Data Challenge (San Francisco). A historical portfolio case study — not affiliated with or endorsed by Lyft, and not a recommendation for current operations.*
