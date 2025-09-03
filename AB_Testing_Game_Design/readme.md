# 🎮 A/B Testing for Game Design – Empowering User Experience & Optimizing Product Recommendations

<img width="2560" height="720" alt="image" src="https://github.com/user-attachments/assets/52ebc043-99fc-44a8-bbc0-29160d3fd44d" />


## 📌 1. Project Overview

### 1.1 Business Context

In the mobile puzzle game **Cookie Cats**, progression gates are introduced to pace gameplay and encourage in-app purchases. This project evaluates the **impact of shifting the first progression gate from level 30 to level 40** on player engagement and retention.  

The **business hypothesis**:  
> By postponing the first gate, players may engage more in the early stages, leading to higher retention and monetization opportunities.

To test this, an **A/B experiment** was conducted on **90,189 players**, randomly assigned to:  

- **Control group (`gate_30`)** – gate appears at level 30  
- **Test group (`gate_40`)** – gate postponed to level 40  

We tracked three core metrics:  
- `sum_gamerounds`: Total game rounds played within 14 days  
- `retention_1`: whether the user returned on Day 1  
- `retention_7`: whether the user returned on Day 7 

---

### 1.2 Statistical Approach (high level)
- **Randomization checks (AA-style):** Welch’s t (engagement), χ² (retention).
- **Method selection:**  
  - Normality (Shapiro–Wilk) → **fail** → non-parametric for engagement.  
  - Variance (Levene) assessed when relevant.  
  - **Engagement:** Mann–Whitney U (median shift, distribution-free).  
  - **Retention:** Two-proportion **z-tests** (binomial model).
- **Uncertainty:** **Bootstrapping (5,000 resamples)** for CI and **directional probability**.
- **Implementation:** reusable `AB_Test()` decision function + figure helpers.

> The goal is to be **assumption-aware** (select the right test for the data) and **decision-oriented** (CI and probability of harm, not just p-values).

---

### 1.3 Strategic Value
- Use A/B testing not only to **find wins** but to **avoid harmful changes**.  
- Make gate policy decisions with a **clear KPI hierarchy** and **guardrails**.  
- Produce artifacts (code + visuals) that can be **reused** for future design experiments.

---

## 📊 2. Data Source & Preprocessing

- [Dataset](https://drive.google.com/file/d/13RZbNDqgYxkW6IBCYpe_lvUKAKaIxkF7/view?usp=sharing): **90,189 rows**, one record per player  
- Fields:  

| Column           | Description                                  |
| ---------------- | -------------------------------------------- |
| `userid`         | Unique player identifier                     |
| `version`        | Group Assignment(`gate_30` / `gate_40`)      |
| `sum_gamerounds` | Total rounds played in first 14 days         |
| `retention_1`    | Returned on Day 1 (Boolean)                  |
| `retention_7`    | Returned on Day 7 (Boolean)                  |

**Group size balance**:  
- `gate_30`: 44,700 players  
- `gate_40`: 45,489 players  

**Outlier Handling**

Basic descriptive statistics revealed a **highly skewed distribution** in `sum_gamerounds`.  
- Median rounds ≈ 16–17  
- Maximum value > **49,000**, suggesting extreme outliers  

Such extreme values can **distort averages** and make hypothesis testing unreliable, as a few hyperactive users disproportionately influence the mean.

To mitigate this, we removed the **single maximum data point** (`sum_gamerounds = 49,854`).  
- After removal, the new maximum = **2,961 rounds**  
- Median and majority distribution remained unchanged  
- Standard deviation was **stabilized**, improving downstream statistical tests  

This adjustment ensures the dataset better reflects **typical player behavior** while avoiding bias from a single anomalous observation.  

📈 *Before removing outliers:*  
<img width="1074" height="212" alt="image" src="https://github.com/user-attachments/assets/fdc1d361-309c-4511-8051-17a0ac6eac54" />
  

📉 *After removing outliers:*  
<img width="804" height="291" alt="image" src="https://github.com/user-attachments/assets/0108f29f-ed46-4549-9c92-3eb188600d46" />
 

> **Insight:** The cleaned dataset better reflects real-world player engagement patterns and avoids drawing misleading conclusions from rare but extreme cases.
 

---

## 🔍 3. Exploratory Data Analysis (EDA)

### 3.1 Game Rounds (Engagement Metric)

We first analyzed whether postponing the first gate affected how much users played during the first 14 days. The analysis combined both **descriptive statistics** and **visual inspection**.

Key findings from the distributions:  
- **Central tendency:** Median values were nearly identical – ~17 rounds for `gate_30` vs ~16 rounds for `gate_40`.  
- **Distributional shape:** Both groups displayed **right-skewed patterns**, indicating that most players engaged moderately, while a small subset played excessively more.  
- **Tail behavior:** Even after outlier removal, the **long tails persisted**, confirming that heavy players exist in both groups but are not systematically biased toward one version.  
- **Overall impression:** No meaningful difference in engagement emerged between the two versions.  

📈 *Game round distributions (full scale):*  
<img width="1081" height="178" alt="image" src="https://github.com/user-attachments/assets/20a32375-1390-40a9-afbf-b2f366bf0052" />


🔍 *Zoomed-in distribution (top 200 rounds):*  
<img width="1072" height="181" alt="image" src="https://github.com/user-attachments/assets/dde23ed3-91d4-4900-b541-e04ba46b2e05" />


> **Insight:** Despite shifting the first gate, both groups exhibited **very similar playtime patterns**. The change neither increased nor decreased overall engagement, suggesting that gate placement primarily influences **retention** rather than **short-term play volume**.


---

### 3.2 Retention
- **Day-1**: 44.8% (`gate_30`) vs 44.2% (`gate_40`)  
- **Day-7**: 19.0% vs 18.2% (noticeable drop in test group)  

| Version | Users | Day-1 Retention | Day-7 Retention | Total Game Rounds |
| ------- | ----- | --------------- | --------------- | ----------------- |
| gate_30 | 44,700 | 0.4482          | 0.1902          | 2,344,795         |
| gate_40 | 45,489 | 0.4423          | 0.1820          | 2,333,530         |

📊 *Day-1 & Day-7 retention rate comparison:*  
<img width="1067" height="406" alt="image" src="https://github.com/user-attachments/assets/d9220c88-0563-4b5b-9e6c-15d2266abc94" />
 

> **Insight:** The gate shift did **not boost engagement**, but **negatively impacted 7-day retention**.

---


## 📐 4. Hypothesis Testing

### 4.0 Decision Flow (Method Selection)
We used a **data-adaptive** testing strategy:

<img width="606" height="488" alt="hypothesis_test_flowchart" src="https://github.com/user-attachments/assets/2e82b537-cd30-4e60-8272-15693b9a0213" />


1) **Normality (Shapiro–Wilk)** → both groups **fail** normality  
2) **Homogeneity (Levene)** → **satisfied**  
3) **Choice** → **Mann–Whitney U** for `sum_gamerounds`; **two-proportion z-tests** for retention

> **(Notebook detail)** `AB_Test()` also computes **Cohen’s d** (pooled) and **observed power** for parametric branches, and auto-plots group distributions.


---

### 4.1 Engagement — `sum_gamerounds` (Numerical Metric)

- Distribution is **non-normal & heavily skewed** → **Mann–Whitney U**
- **p = 0.0509** → ❌ **not significant** (α = 0.05)
- Descriptives: mean 51.34 vs 51.30; pooled **Cohen’s d ≈ 0.0004** (negligible)

> **Interpretation.** The gate delay **does not** impact total play volume.

---

### 4.2 Retention Rates (Binary Metrics)

Two-proportion **z-tests**:

| Metric         | gate_30 | gate_40 | Δ (B–A) | p-value | Conclusion |
|----------------|--------:|--------:|--------:|--------:|-----------|
| **Day-1**      | 0.4482  | 0.4423  | –0.0059 | 0.0739  | ❌ Not significant |
| **Day-7**      | 0.1902  | 0.1820  | –0.0082 | 0.0016  | ✅ Significant drop |

> **Interpretation.** Short-term retention stays flat; **long-term retention deteriorates** under `gate_40`.

---

## 🎲 5. Bootstrapping

### 5.1 Why bootstrap?
P-values don’t show **effect magnitude** or **uncertainty spread**. We therefore **resample with replacement** to obtain empirical **CIs** and **directional probabilities**.

### 5.2 Setup
- Iterations: **5,000** (notebook包含 500 与 5,000 两版，最终取 5,000)  
- For each resample, compute retention rates by group and **Δ = gate_40 – gate_30**  
- Plot distributions and mark **95% CIs**

**Bootstrap histograms (Δ)**  
<img width="806" height="347" alt="image" src="https://github.com/user-attachments/assets/e4b86d87-0540-4226-acbe-cb965a97a440" />


### 5.3 Results
| Metric       | 95% CI              | Excludes 0? | Conclusion                  |
|--------------|---------------------|-------------|-----------------------------|
| Day-1        | **[-0.0123, 0.0005]** | ❌ No        | Not significant             |
| Day-7        | **[-0.0134, -0.0032]** | ✅ Yes       | Significant negative effect |

**Directional probabilities**  
- P(Day-1 diff < 0) = 96.6%  
- P(Day-7 diff < 0) = 99.98%  

📈 *Bootstrap distribution of Day-1 and Day-7 differences:*  
<img width="1051" height="444" alt="image" src="https://github.com/user-attachments/assets/999dc86b-f18f-42fe-b453-e316332083cb" />

> **Takeaway.** Near-certainty that `gate_40` is **worse** for Day-7 retention.

---

## 🧠 6. Insights & Recommendation

### 6.1 Key Findings (What the data says)

- **Engagement didn’t move.** Shifting the first gate **30 → 40** did **not** change total play volume in the first 14 days (`sum_gamerounds`).
- **Short-term retention unchanged.** Day-1 retention shows a small, non-significant dip (–0.59 pp; **p = 0.0739**).
- **Long-term retention worsened.** Day-7 retention **drops ~0.82 percentage points** (19.02% → 18.20%), roughly **–4.3% relative**, and is **statistically significant**:

> **Bottom line:** The gate delay does **not** improve engagement and **hurts Day-7 stickiness** with high statistical confidence.

---

### 6.2 Practical Significance (Scale to Business)

Let `Δ7 = –0.0082` (–0.82 pp). Expected retained-user impact by monthly new installs:

| New installs / month | Baseline Day-7 @ 19.02% | With gate_40 @ 18.20% | **Δ retained users** |
|---:|---:|---:|---:|
| 100,000 | 19,020 | 18,200 | **–820** |
| 500,000 | 95,100 | 91,000 | **–4,100** |
| 1,000,000 | 190,200 | 182,000 | **–8,200** |

Uncertainty from bootstrap 95% CI:
- Best case (–0.32 pp): **–3,200** / 1M installs
- Worst case (–1.34 pp): **–13,400** / 1M installs

> **Revenue lens (plug-in):** `Loss ≈ NewInstalls × (–Δ7) × LTV_per_D7`.  
> Example: if incremental LTV per Day-7 retained user = \$2, loss at 1M installs ≈ \$6.4k–\$26.8k/month.

---

### 6.3 Decision Rationale (Why we won’t ship gate_40)

1) **No upside** on engagement  
2) **Clear downside** on Day-7 retention (statistical & practical)  
3) **Risk-adjusted** view from bootstrap keeps the likely effect **≤ 0**

**✅ Recommendation:** **Do _not_ roll out `gate_40`.** Keep the first gate at level 30.

---

### 6.4 What to Try Next (Actionable roadmap)

- **A/B/n:** test **level 25/30/35/40** to find optimum  
- **Soft-gating:** one-time pass/reward at level 30 to reduce perceived friction  
- **Personalized gating:** gate position based on early-day engagement  
- **UX experiments:** copy/prompts/reward strength to reduce drop-off  
- **Heterogeneity:** slice by region/device/cohort/payer vs. non-payer

---

### 6.5 Rollout & Monitoring Plan (if reconsidered later)

1. **Canary** 1–5% traffic for 2 weeks  
2. **Promotion criteria:** Day-7 Δ ≥ 0 and guardrails neutral  
3. **Post-ship monitoring:** rolling-window CUSUM/SPRT on retention & crash rate  
4. **Automatic rollback:** trigger on any guardrail violation

