# 🎮 A/B Testing for Game Design – Empowering User Experience & Optimizing Product Recommendations

## 📌 1. Project Overview

In the mobile puzzle game **Cookie Cats**, progression gates are introduced to pace gameplay and encourage in-app purchases. This project evaluates the **impact of shifting the first progression gate from level 30 to level 40** on player engagement and retention.  

The **business hypothesis**:  
> By postponing the first gate, players may engage more in the early stages, leading to higher retention and monetization opportunities.

To test this, an **A/B experiment** was conducted on **90,189 players**, randomly assigned to:  

- **Control group (`gate_30`)** – gate appears at level 30  
- **Test group (`gate_40`)** – gate postponed to level 40  

We tracked three core metrics:  
- `sum_gamerounds`: Total game rounds played within 14 days  
- `retention_1`: Day-1 retention  
- `retention_7`: Day-7 retention  

---

## 📊 2. Data Source & Preprocessing

- Dataset: **90,189 rows**, one record per player  
- Fields:  

| Column           | Description                                  |
| ---------------- | -------------------------------------------- |
| `userid`         | Unique player identifier                     |
| `version`        | Experiment group (`gate_30` / `gate_40`)     |
| `sum_gamerounds` | Total rounds played in first 14 days         |
| `retention_1`    | Returned on Day 1 (binary)                   |
| `retention_7`    | Returned on Day 7 (binary)                   |

**Group size balance**:  
- `gate_30`: 44,700 players  
- `gate_40`: 45,489 players  

**Outlier handling**:  
- Maximum game rounds = 49,854 (extreme outlier)  
- Removed top outlier → new max = 2,961 rounds  
- Stabilized variance without affecting median (~16–17 rounds)  

📈 *Distribution of `sum_gamerounds` before outlier removal:*  
![Figure 1 – Distribution before cleaning](results/figure1_distribution_raw.png)  

📉 *Distribution after removing outlier:*  
![Figure 2 – Distribution after cleaning](results/figure2_distribution_clean.png)  

---

## 🔍 3. Exploratory Data Analysis (EDA)

### 3.1 Game Rounds
- Both groups showed **right-skewed distributions**  
- Median rounds: ~17 (`gate_30`) vs ~16 (`gate_40`)  
- No major change in gameplay engagement observed  

📊 *Boxplot & Histogram of game rounds by group:*  
![Figure 3 – Game rounds comparison](results/figure3_gamerounds_boxplot.png)  

---

### 3.2 Retention
- **Day-1**: 44.8% (`gate_30`) vs 44.2% (`gate_40`)  
- **Day-7**: 19.0% vs 18.2% (noticeable drop in test group)  

| Version | Users | Day-1 Retention | Day-7 Retention | Total Game Rounds |
| ------- | ----- | --------------- | --------------- | ----------------- |
| gate_30 | 44,700 | 0.4482          | 0.1902          | 2,344,795         |
| gate_40 | 45,489 | 0.4423          | 0.1820          | 2,333,530         |

📊 *Day-1 & Day-7 retention rate comparison:*  
![Figure 4 – Retention comparison](results/figure4_retention_barchart.png)  

> **Insight:** The gate shift did **not boost engagement**, but **negatively impacted 7-day retention**.

---

## 🧪 4. Experiment Design & Validation

### 4.1 Randomization Check (AA Test)

| Metric           | Test           | p-value | Conclusion          |
| ---------------- | -------------- | ------- | ------------------- |
| `sum_gamerounds` | Welch’s t-test | 0.949   | ✅ Balanced         |
| `retention_1`    | Chi-square     | 0.075   | ✅ Balanced         |
| `retention_7`    | Chi-square     | 0.064   | ✅ Balanced         |

📊 *ECDF of total game rounds:*  
![Figure 5 – ECDF comparison](results/figure5_ecdf.png)  

---

## 📐 5. Hypothesis Testing

### 5.1 Engagement (Game Rounds)
- Distribution highly non-normal → used **Mann-Whitney U test**  
- Result: **p = 0.0509** → ❌ not significant  

### 5.2 Retention (Proportion Tests)

| Metric        | gate_30 | gate_40 | p-value | Conclusion           |
| ------------- | ------- | ------- | ------- | -------------------- |
| Day-1 Ret.    | 44.8%   | 44.2%   | 0.0739  | ❌ No difference      |
| Day-7 Ret.    | 19.0%   | 18.2%   | 0.0016  | ✅ Significant drop   |

---

## 🎲 6. Bootstrapping Validation

- Resampled **5000 iterations** with replacement  
- Calculated Day-1 & Day-7 retention differences  

| Metric        | 95% CI          | Excludes 0? | Conclusion               |
| ------------- | --------------- | ----------- | ------------------------ |
| Day-1 Ret.    | [-0.0123, 0.0005] | ❌ No       | Not significant          |
| Day-7 Ret.    | [-0.0134, -0.0032] | ✅ Yes      | Significant negative     |

**Probabilities:**  
- P(Day-1 diff < 0) = 96.6%  
- P(Day-7 diff < 0) = 99.98%  

📈 *Bootstrap distribution of Day-1 and Day-7 differences:*  
![Figure 6 – Bootstrap CI](results/figure6_bootstrap.png)  

---

## 📌 7. Key Findings & Business Impact

### 7.1 Findings
- 🚫 No improvement in game rounds (engagement)  
- 🚫 No difference in Day-1 retention  
- ⚠️ **Significant 5% relative drop in Day-7 retention** (~0.8pp decrease)  
- Bootstrap confirmed robustness of results  

### 7.2 Business Impact
- Gate shift **reduced long-term retention** → higher churn risk  
- Prevented a harmful design rollout that could have cost **retained players & revenue**  
- Demonstrated value of A/B testing for **risk mitigation** as well as optimization  

**Recommendation:**  
> ❌ **Do not roll out gate_40.** Maintain gate at level 30.

---

## ⚙️ 8. Project Workflow

1. **Data loading & cleaning** (remove outlier)  
2. **EDA** (visualizations, summary stats)  
3. **Randomization check (AA test)**  
4. **Hypothesis testing** (t-test, chi-square, z-test, Mann-Whitney)  
5. **Bootstrapping** (confidence intervals & probability estimates)  
6. **Interpretation & recommendation**  

---

## 📂 9. Repository Structure


