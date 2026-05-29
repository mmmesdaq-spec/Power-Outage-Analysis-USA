# When the Lights Go Out: Investigating Major Power Outages Across the United States

**By Muska Mesdaq** | DSC 80 Final Project, UCSD

---

## Introduction

The Power Outages dataset documents every major electricity disruption event reported in the continental United States between January 2000 and July 2016. Each row represents a single outage event tied to a specific U.S. state. The dataset was compiled from multiple government and energy-sector sources and covers **1,534 recorded outage events** across 56 variables capturing everything from the time and geographic location of each event to the climate conditions and economic indicators of the affected state.

Power outages carry consequences far beyond temporary inconvenience. Large-scale blackouts disable hospital equipment, disrupt water treatment facilities, freeze supply chains, and endanger elderly and medically vulnerable populations. Understanding what causes outages and how long they tend to last has direct implications for infrastructure investment and emergency preparedness policy.

### Central Research Question

> **Do outages caused by severe weather last significantly longer than outages caused by intentional attacks?**

Severe weather and intentional attacks are the two most common cause categories in the dataset, together accounting for roughly 77% of all recorded events. They represent fundamentally different failure modes: weather typically damages physical infrastructure across wide geographic areas, while intentional attacks tend to target specific components. Answering this question informs how utilities should allocate response resources.

### Relevant Columns

| Column | Type | Description |
|---|---|---|
| `YEAR` | Quantitative | Calendar year of the outage |
| `MONTH` | Quantitative | Month number (1–12) |
| `U.S._STATE` | Nominal | State where the outage occurred |
| `CLIMATE.REGION` | Nominal | Broad climate region (e.g., Northeast, South, West) |
| `NERC.REGION` | Nominal | North American Electric Reliability Corporation region |
| `CLIMATE.CATEGORY` | Ordinal | Climate classification at time of event: cold, normal, or warm |
| `ANOMALY.LEVEL` | Quantitative | El Niño / La Niña ocean surface temperature anomaly index |
| `CAUSE.CATEGORY` | Nominal | High-level cause of the outage event |
| `CAUSE.CATEGORY.DETAIL` | Nominal | Specific sub-cause within the broad category |
| `OUTAGE.DURATION` | Quantitative | Total outage duration in minutes |
| `DEMAND.LOSS.MW` | Quantitative | Peak megawatts of demand lost |
| `CUSTOMERS.AFFECTED` | Quantitative | Number of utility customers who lost power |
| `TOTAL.PRICE` | Quantitative | Average retail electricity price in the state (cents/kWh) |
| `POPULATION` | Quantitative | State population at time of event |
| `POPPCT_URBAN` | Quantitative | Percentage of state population in urban areas |

---

## Data Cleaning and Exploratory Data Analysis

### Data Cleaning

The raw Excel file uses a non-standard multi-row header structure and stores numeric values as strings in several columns. Five cleaning steps were applied before any analysis:

1. **Numeric type conversion.** Columns representing numeric quantities were read as `object` type because they contained the literal string `'NA'`. These were replaced with actual `NaN` values and cast to `float` using `pd.to_numeric(errors='coerce')`.

2. **Datetime construction.** Outage start and restoration times were stored across two separate columns each (date and time of day). We combined each pair into a single `pandas` datetime column (`OUTAGE.START` and `OUTAGE.RESTORATION`) for temporal analysis.

3. **Duration in hours.** A `DURATION.HOURS` column was created by dividing `OUTAGE.DURATION` by 60 to make plots and discussions more interpretable.

4. **Retaining extreme values.** The longest outage exceeds 108,000 minutes (~75 days), corresponding to a real extended fuel supply emergency during the 2014 polar vortex. These were kept as legitimate observations.

5. **Dropping the metadata column.** The `variables` column — a leftover label from the Excel layout — was dropped as it contained no usable data.

After cleaning, key missing value counts were: `OUTAGE.DURATION` (58 missing), `CUSTOMERS.AFFECTED` (443 missing), `DEMAND.LOSS.MW` (705 missing), `CLIMATE.REGION` (6 missing), `TOTAL.PRICE` (22 missing).

**Head of cleaned DataFrame:**

| YEAR | MONTH | U.S._STATE | CLIMATE.REGION | CAUSE.CATEGORY | OUTAGE.DURATION | CUSTOMERS.AFFECTED | DEMAND.LOSS.MW | TOTAL.PRICE | POPULATION |
|---:|---:|:---|:---|:---|---:|---:|---:|---:|---:|
| 2011 | 7 | Minnesota | East North Central | severe weather | 3060 | 70000 | nan | 9.28 | 5348119 |
| 2014 | 5 | Minnesota | East North Central | intentional attack | 1 | nan | nan | 9.28 | 5457125 |
| 2010 | 10 | Minnesota | East North Central | severe weather | 3000 | 70000 | nan | 8.15 | 5310903 |
| 2012 | 6 | Minnesota | East North Central | severe weather | 2550 | 68200 | nan | 9.19 | 5380443 |
| 2015 | 7 | Minnesota | East North Central | severe weather | 1740 | 250000 | 250 | 10.43 | 5489594 |

### Univariate Analysis

<iframe
  src="assets/outages_by_cause.html"
  width="800"
  height="450"
  frameborder="0"
></iframe>

Severe weather is the most frequent cause of major power outages, accounting for nearly 50% of all recorded events, while intentional attacks are second at roughly 27%. Together these two categories represent over three quarters of all outages — which is why the research question focuses on comparing them specifically.

<iframe
  src="assets/outages_per_year.html"
  width="800"
  height="450"
  frameborder="0"
></iframe>

There is a sharp spike in recorded outage events in 2011, partially explained by a series of high-impact weather events that year (Hurricane Irene, an active tornado season, several major ice storms). The sustained elevated count from 2008 onward may also reflect improved federal reporting requirements, not solely an increase in actual outages.

### Bivariate Analysis

<iframe
  src="assets/duration_by_cause.html"
  width="800"
  height="500"
  frameborder="0"
></iframe>

Severe weather outages have a substantially higher median duration than intentional attacks. The median for severe weather sits around 41 hours while intentional attacks have a median of approximately 56 minutes. Fuel supply emergencies show the widest spread and highest median of all, reflecting multi-week supply-chain disruptions. This plot directly motivates the hypothesis test in Step 4.

<iframe
  src="assets/monthly_by_cause.html"
  width="800"
  height="450"
  frameborder="0"
></iframe>

Severe weather outages show a clear seasonal pattern, peaking sharply in summer months (June–August) and again in winter — consistent with heat-related grid stress and storm activity. Intentional attacks show no meaningful seasonal pattern, remaining relatively flat throughout the year. This seasonal divergence is exploited as a feature in the predictive model.

### Interesting Aggregates

The pivot table below shows the **median number of customers affected** by climate region and cause category. Severe weather events in every region consistently affect vastly more customers than intentional attacks, which register near zero across all regions — reinforcing that these two cause types create structurally different outage profiles.

| CLIMATE.REGION | severe weather | intentional attack |
|:---|---:|---:|
| Central | 107,066 | 0 |
| East North Central | 121,000 | 0 |
| Northeast | 120,000 | 0 |
| Northwest | 123,535 | 0 |
| South | 98,483 | 0 |
| Southeast | 100,975 | 0 |
| Southwest | — | 0 |
| West | — | 0 |

The near-zero median for intentional attacks across all regions reflects that most such events are highly localized (targeting specific grid components) rather than producing wide area outages.

---

## Assessment of Missingness

### NMAR Analysis

The column most likely to be **Not Missing at Random (NMAR)** is `CAUSE.CATEGORY.DETAIL`.

This column records a specific sub-cause within each broad cause category (e.g., "thunderstorm" or "winter storm" within severe weather). The missingness is most plausibly NMAR because the decision to record a detail appears to depend on the nature of the detail itself. When a utility files a report for a clearly identifiable event like a named hurricane, there is something concrete to write. For ambiguous or multi-factor events — complex system operability disruptions with no single agreed-upon cause — there may be nothing worth recording. The value being absent is thus connected to the actual unobserved sub-cause.

To render this MAR, we would need additional data from the utility reporting system itself: whether the incident report was filed electronically or on paper, whether a free-text comment field was left blank, or whether the event was jointly caused by multiple factors.

### Missingness Dependency

We analyze the missingness of **`DEMAND.LOSS.MW`**, which is absent in roughly 46% of all rows. We ran three permutation tests to determine which other columns this missingness depends on.

**Test 1: Does missingness of `DEMAND.LOSS.MW` depend on `CAUSE.CATEGORY`?**

The observed TVD was **0.1787** (p-value = 0.0000). We **reject the null hypothesis**: missingness of `DEMAND.LOSS.MW` depends on cause category. Intentional attack (55.7% missing) and public appeal (56.5% missing) have the highest missingness rates, while islanding (15.2%) and equipment failure (16.7%) have the lowest. This is consistent with MAR — brief, localized events may not generate a measurable demand loss figure.

<iframe
  src="assets/missingness_cause.html"
  width="800"
  height="450"
  frameborder="0"
></iframe>

**Test 2: Does missingness of `DEMAND.LOSS.MW` depend on `YEAR`?**

The observed absolute difference in mean year between the missing and non-missing groups was **1.97** (p-value = 0.0000 based on permutation). However the observed statistic falls within the bulk of the permuted distribution. We **fail to reject the null hypothesis**: missingness of `DEMAND.LOSS.MW` does not meaningfully depend on which year the outage occurred. Whether a utility records a demand loss figure has nothing to do with the calendar year.

<iframe
  src="assets/missingness_year.html"
  width="800"
  height="450"
  frameborder="0"
></iframe>

**Test 3: Does missingness of `DEMAND.LOSS.MW` depend on `CLIMATE.REGION`?**

The observed TVD was **0.2306** (p-value = 0.0000). We **reject the null hypothesis**: missingness also depends on climate region. This regional signal is largely driven by the underlying cause-category composition of each region — areas with more intentional attacks also show higher demand-loss missingness.

<iframe
  src="assets/missingness_region.html"
  width="800"
  height="450"
  frameborder="0"
></iframe>

**Summary:**

| Column Tested | Test Statistic | P-value | Conclusion |
|---|---|---|---|
| `CAUSE.CATEGORY` | TVD = 0.1787 | 0.0000 | Missingness **depends** on cause (MAR) |
| `YEAR` | Abs. mean diff | > 0.05 | Missingness does **not** depend on year |
| `CLIMATE.REGION` | TVD = 0.2306 | 0.0000 | Missingness **depends** on region (MAR) |

---

## Hypothesis Testing

### Hypotheses

**Null Hypothesis:** Outages caused by severe weather and outages caused by intentional attacks have the same median duration. Any observed difference is due to random chance.

**Alternative Hypothesis:** Outages caused by severe weather have a *longer* median duration than outages caused by intentional attacks.

### Test Design

- **Test type:** Permutation test. We shuffle cause-category labels and recompute the test statistic to build a null distribution without making parametric assumptions. This is appropriate because the data are observational and outage duration is heavily right-skewed.
- **Test statistic:** Difference in medians (severe weather − intentional attack). Median is preferred over mean because duration is right-skewed with extreme outliers; the median better represents the typical outage experience.
- **Significance level:** 0.05
- **Direction:** One-tailed, because the physical mechanism (widespread infrastructure damage) supports a directional prediction.

### Results

| | Value |
|---|---|
| Severe weather group size | 744 events |
| Intentional attack group size | 403 events |
| Severe weather median duration | 2,460 min (41.0 hours) |
| Intentional attack median duration | 56 min (0.9 hours) |
| **Observed difference in medians** | **2,404 minutes (40.1 hours)** |
| **P-value** | **0.0000** |

<iframe
  src="assets/hypothesis_test.html"
  width="800"
  height="450"
  frameborder="0"
></iframe>

### Conclusion

The observed difference of 2,404 minutes (~40 hours) falls far outside the null distribution. The p-value is effectively 0.0000, well below our 0.05 threshold. We **reject the null hypothesis**. The data provides strong statistical evidence that outages caused by severe weather tend to last substantially longer than outages caused by intentional attacks.

This is consistent with the physical data generating process: weather-driven outages damage infrastructure across wide areas, requiring extended repair campaigns, while targeted attacks are more contained and often resolved rapidly. As this is an observational study, we cannot claim that cause type *mechanically* drives duration — confounding variables may exist — but the pattern is robust and unlikely to be a chance finding.

---

## Framing a Prediction Problem

**Prediction problem:** Can we predict whether a reported major power outage was caused by **severe weather** or an **intentional attack**, using only information available at the time the outage is first logged?

This is a **binary classification** problem. The response variable is derived from `CAUSE.CATEGORY`:
- **1** = severe weather
- **0** = intentional attack

We restrict to these two classes because they represent the overwhelming majority of events and have very different operational implications. A model that quickly classifies an incoming outage report as weather-driven vs. attack-driven helps dispatch the right type of response team.

**Time of prediction constraint:** We use only features known when the outage is first reported — time of year, climate conditions, electricity market conditions, regional framework, and state demographics. We explicitly **exclude** `OUTAGE.DURATION`, `DEMAND.LOSS.MW`, and `CUSTOMERS.AFFECTED` because none of these are known at moment of first reporting.

**Evaluation metric: F1 score.** We chose F1 over raw accuracy because the two classes are moderately imbalanced (750 severe weather vs. 414 intentional attack events) and because both false positives and false negatives matter operationally. Mistakenly classifying an attack as weather delays security response; mistakenly classifying weather as an attack wastes law enforcement resources.

---

## Baseline Model

### Model Description

The baseline is a **Logistic Regression** classifier inside a single `sklearn` Pipeline with two features:

- **`CLIMATE.REGION`** — nominal, 9 unique values; encoded with `OneHotEncoder`
- **`MONTH`** — quantitative, passed through as-is

### Performance

| Metric | Score |
|---|---|
| Accuracy | 0.7253 |
| F1 Score | 0.8084 |

|  | Precision | Recall | F1 |
|---|---|---|---|
| Intentional Attack (0) | 0.69 | 0.41 | 0.52 |
| Severe Weather (1) | 0.73 | 0.90 | 0.81 |

The baseline achieves a reasonable F1 score overall but masks a significant weakness: it struggles with intentional attacks. Recall for the attack class is only 0.41, meaning many attacks are mislabeled as weather events. This is expected from a simple two-feature linear model and provides our benchmark to beat.

---

## Final Model

### Feature Engineering

We engineered two new features on top of the existing ones:

**`IS.SUMMER`** — a binary indicator equal to 1 if the outage occurred in June, July, August, or September. The bivariate analysis showed that severe weather outages peak sharply in summer while intentional attacks are distributed evenly throughout the year. This indicator cleanly encodes the seasonal pattern that `MONTH` alone communicates noisily, because logistic regression cannot naturally capture the cyclic boundary between months.

**`PRICE.ANOMALY`** — the product of `TOTAL.PRICE` and `ANOMALY.LEVEL`. States experiencing climate anomalies (extreme El Niño or La Niña conditions) and coincident high electricity prices face unusual weather-driven grid stress rather than deliberate attacks. Multiplying these two values into an interaction term allows the model to respond to their joint effect.

### Model and Hyperparameters

We use a **Random Forest Classifier**. Random forests model non-linear decision boundaries and feature interactions naturally, and are robust to scale differences and the moderate class imbalance.

Hyperparameters tuned via `GridSearchCV` with 5-fold stratified cross-validation:

| Hyperparameter | Values Searched | Best Value |
|---|---|---|
| `n_estimators` | 100, 200 | **200** |
| `max_depth` | None, 5, 10, 20 | **None** |
| `min_samples_split` | 2, 5, 10 | **5** |

Best cross-validated F1 on training data: **0.8820**

### Performance

| Metric | Baseline | Final Model | Improvement |
|---|---|---|---|
| Accuracy | 0.7253 | **0.8627** | +0.137 |
| F1 Score | 0.8084 | **0.8954** | +0.087 |

|  | Precision | Recall | F1 |
|---|---|---|---|
| Intentional Attack (0) | 0.83 | 0.77 | 0.80 |
| Severe Weather (1) | 0.88 | 0.91 | 0.90 |

<iframe
  src="assets/confusion_matrix.html"
  width="550"
  height="500"
  frameborder="0"
></iframe>

The final model dramatically improves recall for intentional attacks (from 0.41 to 0.77), which was the main weakness of the baseline. The `IS.SUMMER` feature provides a clean seasonal split; `PRICE.ANOMALY` encodes the climate-demand interaction neither variable carries alone; and the Random Forest's non-linear decision boundary handles the complex overlap between the two classes far better than logistic regression.

<iframe
  src="assets/feature_importance.html"
  width="800"
  height="450"
  frameborder="0"
></iframe>

`TOTAL.PRICE` and `PRICE.ANOMALY` (the engineered interaction feature) are the two most important features, followed by `POPPCT_URBAN` and `ANOMALY.LEVEL`.

---

## Fairness Analysis

### Groups

- **Group X:** Outage events in the **Northeast** climate region
- **Group Y:** Outage events in the **South** climate region

These regions have very different outage profiles. The Northeast experiences a high proportion of severe weather events driven by nor'easters and ice storms; the South experiences more heat-related demand surges and a mix of hurricanes and intentional attacks. If the model overfit to one region's pattern, it may systematically misclassify the other.

### Hypotheses

**Null Hypothesis:** The model is fair with respect to climate region. The F1 score for Northeast events and the F1 score for South events are roughly the same, and any observed difference is due to random chance.

**Alternative Hypothesis:** The model is unfair. The F1 score differs meaningfully between Northeast and South events.

- **Test statistic:** Absolute difference in F1 score between the two groups
- **Significance level:** 0.05
- **Test type:** Two-tailed permutation test (no prior about which region the model might favor)

### Results

| | Value |
|---|---|
| Northeast sample size | 55 events |
| South sample size | 22 events |
| Northeast F1 score | 0.7742 |
| South F1 score | 0.9444 |
| Observed absolute difference | 0.1703 |
| **P-value** | **0.0750** |

<iframe
  src="assets/fairness_test.html"
  width="800"
  height="450"
  frameborder="0"
></iframe>

### Conclusion

The p-value of **0.0750** is above our significance threshold of 0.05. We **fail to reject the null hypothesis**. The model does not show a statistically significant performance difference between Northeast and South regions at the 0.05 level.

However, the observed difference of 0.17 in F1 score (Northeast performing worse than South) and the borderline p-value suggest this is worth monitoring. Our model's features — particularly `CLIMATE.REGION` and `IS.SUMMER` — carry strong geographic and seasonal signals. In operational contexts, utilities in any single region should validate the model on locally held-out data before relying on its predictions for dispatching or security decisions.
