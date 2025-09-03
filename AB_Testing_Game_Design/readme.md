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
- `retention_1`: whether the user returned on Day 1  
- `retention_7`: whether the user returned on Day 7 

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

### 4.1 Game Rounds Played (Numerical Metric)
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


