# Early Driver Disengagement: Designing a Risk-Based Retention Experiment

*A case study using the 2016 Lyft San Francisco driver dataset, framed and executed as a business decision rather than only a modeling exercise.*

---

## Executive Summary

By a driver's 14th day, two simple signals can identify elevated risk of future inactivity:

1. How many days the driver was active during the first two weeks
2. How long the driver has already been idle

A transparent two-signal approach performs almost as well as an 11-variable model. The practical answer is a simple, explainable risk screen followed by a controlled test, not a black-box model.

The deliverable is a pilot design and a capacity view for allocating limited outreach. It is not a churn score, and it does not manufacture an ROI estimate.

> **Headline:** Drivers active on 4-7 of their first 14 days represent about one-third of eligible drivers but nearly half of all subsequently observed inactivity events. This makes them an efficient population in which to test retention outreach.

---

## The question

Reaching out to new drivers costs time and money, so Lyft needs a disciplined way to focus limited attention.

> **Can a driver's first two weeks of activity identify who is likely to go quiet, early enough and clearly enough to support a retention experiment?**

This is a focus-and-learn decision, not a prediction contest.

Risk indicates who may become inactive. Only a controlled experiment can establish who responds to outreach and whether targeted outreach is better than a broader approach.

---

## Why this framing, rather than driver lifetime value

The Lyft driver dataset is commonly used for a lifetime-value analysis. This project reframes it as an early-retention and marketplace-supply problem.

The analysis remains strict about what observational data can and cannot prove. The outcome is a defensible experiment design rather than a churn dashboard or a speculative driver-value calculation.

---

## Data and audit: trust earned, not assumed

| File | Description | Rows |
|---|---|---:|
| `driver_ids.csv` | One row per driver, including onboarding date | 937 |
| `ride_ids.csv` | One row per completed ride, including distance, duration, and prime time | 193,502 |
| `ride_timestamps.csv` | One row per ride step, from request through drop-off | 970,405 |

Key audit findings:

- **The largest weakness is join integrity, not missing values.** Only 854 of 937 driver IDs overlap between the onboarding and ride-detail files. There are 83 IDs unique to each file, plus approximately 4.5% two-sided slippage at the ride level.
- **The timeline-usable population contains 837 drivers.** Of these, 834 are eligible at the day-14 landmark because they have at least one timestamped ride before the prediction point.
- **The 83 onboarding-only IDs are not labeled as drivers who never activated.** Their provenance is unresolved, so they are treated as a data-quality limitation rather than automatically classified as disengaged.
- **Ride duration reconciles exactly.** For every matched ride, the recorded duration equals the difference between pickup and drop-off timestamps.
- **UTC is the primary clock.** Interpreting onboarding as San Francisco midnight creates 177 rides across 66 drivers that appear to occur before onboarding. UTC produces zero such contradictions.
- **Population loss is not random.** Completeness declines across later onboarding cohorts, and the 17 matched drivers without usable timestamps have lower ride volume. Results are therefore scoped to fully linkable drivers. The direction of any remaining selection bias is unknown.

---

## Approach: from a fuzzy question to a defensible decision

1. **Frame the decision:** define the owner, business choice, permitted claims, and prohibited claims.
2. **Structure the problem:** establish sequential decision gates covering data trust, materiality, early signal, model complexity, concentration, stability, and actionability.
3. **Establish honest data:** audit joins, timestamps, anomalies, follow-up, and the analytical population.
4. **Analyze:** compare early activity signals, survival timing, a simple benchmark, and a more complex challenger.
5. **Find the business consequence:** identify an experiment population, quantify capacity trade-offs, and define what a pilot must prove.
6. **Communicate:** summarize the decision in this README and the [executive memo](EXECUTIVE_MEMO.md).

### Outcome definition

The primary outcome is the first time a driver reaches 21 consecutive days without a ride after the day-14 landmark.

It is deliberately not called churn. Approximately 35% of drivers who reach the 21-day inactivity threshold are subsequently observed returning within another three weeks.

Thresholds of 14, 21, and 28 days are treated as different business outcomes:

- 14 days captures more temporary breaks
- 21 days is the primary prolonged-inactivity definition
- 28 days represents more severe inactivity

### No-leakage rule

Every targeting signal uses only information available before the day-14 landmark.

The driver's current inactivity gap at day 14 is a valid predictor because it is known at the decision point. The event occurs when the 21-day threshold is crossed; it is never calculated backward from the driver's last observed ride.

---

## Key results

### 1. Lower early activity is associated with faster and more frequent inactivity

![Inactivity curves by early-activity group](chart2.png)

### 2. Near-term inactivity falls from 34% to 2% as early activity becomes consistent

![Near-term inactivity by early-activity group](chart1.png)

| First-14-day activity | Drivers | Share of observed inactivity events | 28-day inactivity probability |
|---|---:|---:|---:|
| 1-3 active days | 100 | 21% | 34.0% |
| **4-7 active days** | **269** | **49%** | **26.0%** |
| 8-10 active days | 209 | 19% | 13.9% |
| 11-14 active days | 256 | 12% | 2.3% |

The 4-7-day segment combines elevated risk with evidence of meaningful initial participation. The 1-3-day segment has higher risk but represents substantially less observed early ride activity.

### 3. Nine additional variables add only 0.003 discrimination

![Model comparison](images/chart3.png)

| Approach | Validated C-index* | Interpretation |
|---|---:|---|
| Active-day rule, 1 signal | 0.712 | Strong standalone signal |
| Current-gap rule, 1 signal | 0.708 | Strong secondary signal |
| Two-signal model | 0.732 | Best practical balance |
| 11-variable model | 0.735 | Rejected; 0.003 does not justify nine additional inputs |

*\*A C-index of 0.50 represents random ranking. Among comparable driver pairs, a C-index of approximately 0.71-0.73 means the approach correctly orders which driver becomes inactive sooner about 71%-73% of the time. It measures ranking, not calibrated individual probability.*

The full model is fractionally more discriminating, but the gain is too small to justify the additional complexity. The operational recommendation is to use activity tiers first and current inactivity gap as a within-tier prioritization signal.

### 4. Prioritizing 30% of drivers covers approximately half of observed inactivity events

![Capacity curve](images/chart4.png)

| Share of drivers prioritized | Share of observed events covered |
|---:|---:|
| 10% | 18.7% |
| 20% | 34.9% |
| 30% | 51.4% |
| 40% | 64.1% |
| 50% | 75.4% |

This is a capacity curve, not an optimized cutoff. Selecting a final outreach level requires intervention cost, operational capacity, treatment responsiveness, and the value of incremental activity.

The same risk-ranked 30% represents only approximately 11% of early rides. This is the central business tension: concentrated risk does not automatically imply concentrated value or responsiveness.

---

## Recommendation

Run a controlled outreach experiment at the day-14 landmark.

### Primary confirmatory population

Start with drivers active on 4-7 of their first 14 UTC tenure days.

Historically, this population:

- Represents 32.3% of eligible drivers
- Contains 48.9% of all subsequently observed inactivity events
- Represents 17.8% of early rides
- Has a 26.0% probability of crossing the inactivity threshold within the following 28 days

### Experiment structure

- Randomize eligible drivers 1:1 between standardized outreach and business as usual.
- Use a fixed 28-day primary measurement window after the day-14 landmark.
- Measure the intention-to-treat difference in 21-day inactivity crossings.
- Stratify randomization by current inactivity gap: under 3 days, 3-7 days, and 7-14 days.
- Include smaller randomized samples from other activity levels. Without comparison strata, the experiment cannot determine whether risk-based prioritization is better than broader outreach.
- Track secondary outcomes such as active days, completed rides, treatment delivery, opt-outs, complaints, and outreach cost.

### Interim rule, low-cost use only

If low-cost, reversible outreach must begin before pilot results are available, prioritize drivers in the 4-7-day segment who have already been idle for at least 3 days at the day-14 landmark.

Historically, approximately 39% of this priority group crosses the inactivity threshold within the following 28 days.

This is a temporary prioritization heuristic. It is not evidence for expensive incentives or automatic treatment allocation.

---

## Technical validation

### The broad 1-7-day screen separates risk in every onboarding cohort

![Risk separation across onboarding cohorts](images/chart5.png)

This chart validates the broader activity screen, not the narrower 4-7-day confirmatory pilot population.

Within every onboarding cohort, drivers active on no more than 7 of their first 14 tenure days have a higher observed event proportion than drivers active on more than 7 days.

Later cohorts have fewer observed events and shorter follow-up. Absolute event rates should therefore not be compared directly across cohorts.

---

## What this project does not claim

This is observational, single-cohort, 2016 data. It supports identifying elevated risk and designing an experiment.

It does not establish that:

- Outreach prevents inactivity
- Outreach saves rides or generates ROI
- High-risk drivers are the most responsive or reachable
- Risk-based prioritization is superior to uniform outreach
- Incentives are justified
- The findings apply to Lyft's current marketplace

Going quiet is treated as a re-engagement signal, not confirmed churn.

---

## Reproduce the analysis

```bash
git clone <this-repo>
cd <this-repo>
pip install -r requirements.txt

# Open the notebook and run all cells from top to bottom
