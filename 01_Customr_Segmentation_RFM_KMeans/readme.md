# 👥 Customer Segmentation with RFM & K-Means Clustering

---

## 1. Project Overview

### 1.1 Business Context

**Lucky & Me**, a children’s performance underwear brand, faces several operational challenges common in retail:

- **Low in-store traffic**, affecting conversion and visibility.
- **Inventory mismatches**, such as out-of-stock and missing sizes, hurting customer experience.
- **Online transaction obstacles**, including lack of attribution and sales tracking.
- **Limited marketing resources**, especially for digital content and personalized outreach.

To address these pain points, this project applies customer segmentation to enable data-driven marketing and strategic CRM improvement.


### 1.2 Objective

This project aims to segment customers using historical transaction data and design actionable marketing strategies that:

- Enhance customer retention and loyalty,
- Boost repeat purchase rates,
- Optimize promotional resource allocation.

We use:

- **RFM analysis** (Recency, Frequency, Monetary value) to derive customer value features,
- **K-Means clustering** to uncover behavioral segments,
- **Elbow method and silhouette score** to validate cluster quality,
- **Visual analytics** to communicate insights and drive business decisions.


### 1.3 Strategic Value

#### 💡 Why Segment Customers?

Customer segmentation enables tailored strategies based on customer behavior and value:

- **VIPs** → Exclusive discounts, loyalty perks, and early access.
- **Regulars** → Rewards programs and personalized coupons.
- **New users** → Welcome bundles and first-purchase incentives.
- **At-risk/inactive** → Re-engagement offers and periodic check-ins.

This empowers Lucky & Me to implement **tiered marketing strategies** that increase lifetime value and reduce churn.


### 1.4 Technical Approach

- **Feature Engineering**:
  - Constructed RFM metrics from raw transaction logs.
- **Clustering**:
  - Created 8 RFM-based tiers.
  - Used **K-Means** to identify 3 core customer groups.
- **Validation**:
  - Applied the **elbow method** and **silhouette scores**.
- **Visualization**:
  - RFM heatmaps, cluster plots, and customer tier dashboards.
 

### 1.5 Tools & Technologies

- **Languages**: Python (Pandas, NumPy, Sklearn, Matplotlib, Seaborn)
- **Algorithms**: K-Means, DBSCAN
- **Evaluation**: Silhouette Score, Elbow Plot
- **Visualization**: Heatmaps, scatter plots, segment profiling
- **Strategy Layer**: Lifecycle-based marketing recommendations

---

## 2. Data Preprocessing

### 2.1 Dataset Overview

The [dataset](https://drive.google.com/file/d/1lP5kFSBI0iLdg-mAF27s9a8MjyR07FVT/view?usp=drive_link) covers transactions between Dec 1, 2010, and Dec 9, 2011, and includes:

- ~500,000 transactions  
- ~4,000 valid customers (after cleaning)

| Column       | Description                                           |
|--------------|-------------------------------------------------------|
| InvoiceNo    | Transaction ID (prefix 'C' = cancelled)               |
| StockCode    | Product code                                          |
| Description  | Product description                                   |
| Quantity     | Quantity purchased                                    |
| InvoiceDate  | Timestamp of transaction                              |
| UnitPrice    | Price per item (GBP)                                  |
| CustomerID   | Unique customer ID                                    |
| Country      | Customer location                                     |


### 2.2 Missing Data Removal

We first examined overall data completeness to ensure the dataset was suitable for RFM feature engineering and clustering.

Using `df.info()` and a `missingno.matrix` visualization, we observed that:

- Core transactional fields (`InvoiceNo`, `StockCode`, `Quantity`, `InvoiceDate`, `UnitPrice`, `Country`) showed almost no missing values.
- `Description` had minor missingness (~1%), reflecting a small number of transactions without product names.
- `CustomerID` exhibited substantial missingness (~25%), indicating guest/anonymous purchases or records without captured identifiers.

<img width="1078" height="461" alt="image" src="https://github.com/user-attachments/assets/4a140210-e669-40d3-9086-67b218350ec9" />


**Insight:** The dataset is largely complete for modeling. However, rows without `CustomerID` should be excluded from RFM/K-Means to avoid instability, while the anonymous segment can be analyzed separately as a **“Guest”** cohort for marketing.


### 2.3 Missing Data Removal

We then evaluated cancellations/returns and other invalid lines, which can bias Monetary values and distort downstream clustering.

Using invoice patterns and basic validity checks, we observed that:

- Credit notes/cancellations could be identified by `InvoiceNo` containing **“C”**, and negative quantities reflected returns rather than purchases.
- Non-positive prices or quantities (e.g., `UnitPrice ≤ 0`, `Quantity ≤ 0`) represented corrections, freebies, or data-entry artifacts rather than revenue-bearing transactions.
- After removing credit notes and keeping only positive quantities and prices, distribution tails shrank and summary statistics stabilized, with negligible impact on the vast majority of records.

  <img width="908" height="127" alt="image" src="https://github.com/user-attachments/assets/1fb1e292-aa11-4de9-8238-e4f360ea02f9" />


**Insight:** Filtering cancellations/returns and non-positive lines materially improves the reliability of Monetary features, reducing noise before RFM computation and K-Means clustering.

---

## 3. RFM Analysis

In this project, we applied two different approaches to RFM-based customer segmentation:

**1. Rule-Based RFM (8 Segments)**  
- Each customer is classified by comparing their Recency (R), Frequency (F), and Monetary (M) values against the dataset mean.  
- Scores above the mean are marked as `1` and below as `0`, which produces `2 × 2 × 2 = 8` possible combinations.  
- This creates classic RFM categories such as:  
  - **High-Value Customers (R↓F↑M↑)**  
  - **Retention Customers (R↓F↑M↓)**  
  - **Churn-Risk Customers (R↑F↓M↓ / R↑F↓M↑)**  
  - **Developing Customers (R↑F↑M↓ / R↑F↓M↓)**  
- ✅ Strength: clear interpretability and alignment with textbook RFM analysis.  
- ⚠️ Limitation: rigid binarization may over-split customers, and managing 8 segments can be too complex for real-world marketing execution.  

**2. K-Means Clustering (4–5 Segments)**  
- Instead of predefined thresholds, K-Means clustering groups customers based on the natural patterns in their RFM values.  
- After evaluating with the Elbow Method and Silhouette Scores, the optimal cluster number was found to be **k=4 or k=5**.  
- Example clusters:  
  - **Inactive / Low-Value Customers**  
  - **Mid-Value Loyal Customers**  
  - **High-Value Customers**  
  - **VIP Customers**  
  - *(Optional, in k=5)* **New / Moderate Buyers**  
- ✅ Strength: fewer, more actionable segments that better reflect customer behavior.  
- ⚠️ Limitation: requires interpretation of clusters, since labels are not predefined.  

---


### 3.1 Building the RFM Table

We began by aggregating customer-level data using a pivot table:

```python
rfm = df.pivot_table(
    index='customerid',
    values=["invoiceno", "totalcost", "invoicedate"],
    aggfunc={
        "invoiceno": pd.Series.nunique,
        "totalcost": "sum",
        "invoicedate": "max"
    }
)
```

- invoiceno → **Frequency:** Number of unique purchases 
- totalcost → **Monetary:** Total expenditure  
- invoicedate → Used to compute **Recency:** days since last purchase


We defined **Recency** as the number of days since a customer's most recent transaction:

```python
rfm['Recency'] = (rfm.invoicedate.max() - rfm.invoicedate) / np.timedelta64(1, 'D')
```

We also renamed columns:

```python
rfm.rename(columns={"invoiceno": "Frequency", "totalcost": "Monetary"}, inplace=True)
```

This resulted in an RFM table where each row represents one customer:

<img width="429" height="208" alt="image" src="https://github.com/user-attachments/assets/9921d645-9d87-4088-a609-ad59918a5b12" />



### 3.2 Labeling via Thresholds - Rule-Based RFM

We normalized each RFM feature and binarized it based on whether it was above or below the mean. This created a 3-bit string (e.g., "011"), which was mapped to a customer label:

```python
def rfm_func(x):
    level = x.apply(lambda x:'1'if x>0 else '0')
    level = level.Recency + level.Frequency + level.Monetary
    d = {
        "011": "High-Value Customers (R↓F↑M↑)",
        "111": "Important Retention Customers (R↑F↑M↑)",
        "001": "Important Churn-Risk Customers (R↓F↓M↑)",
        "101": "Important Developing Customers (R↑F↓M↑)",
        "110": "General Value Customers (R↑F↑M↓)",
        "010": "General Retention Customers (R↓F↑M↓)",
        "100": "General Churn-Risk Customers (R↑F↓M↓)",
        "000": "General Developing Customers (R↓F↓M↓)"
    }
```

## 🎯 RFM Segments and Suggested Marketing Strategies

| Segment Type                     | R   | F   | M   | Marketing Focus |
|----------------------------------|-----|-----|-----|-----------------|
| **High-Value Loyal Customers**   | ↑   | ↑   | ↑   | VIP programs, exclusive discounts, early access to new products. |
| **High-Value Retention Customers** | ↓   | ↑   | ↑   | Win-back campaigns, personalized offers to re-engage. |
| **High-Potential Customers**     | ↑   | ↓   | ↑   | Encourage repeat purchases with targeted promotions. |
| **At-Risk High Spenders**        | ↓   | ↓   | ↑   | Special offers to prevent churn, personalized follow-up. |
| **Value Growth Customers**       | ↑   | ↑   | ↓   | Upselling and cross-selling to increase spending. |
| **Retention-Focused Customers**  | ↓   | ↑   | ↓   | Maintain relationship with loyalty rewards and incentives. |
| **New or Developing Customers**  | ↑   | ↓   | ↓   | Onboarding offers, nurture campaigns to build loyalty. |
| **Low-Value Inactive Customers** | ↓   | ↓   | ↓   | Low-cost mass marketing, reactivation campaigns. |



### 3.3 K-Means Clustering

RFM segmentation provides a powerful way to classify customers based on Recency, Frequency, and Monetary value. However, assigning customers into fixed categories via manual RFM score thresholds can oversimplify the underlying customer behaviors. To uncover **natural groupings** in the data without predefined labels, we implemented **K-Means clustering**, a widely used technique for partitioning observations into cohesive groups based on feature similarity. The goal was to let the data drive the segmentation.


### 3.4 Choosing the Optimal K

A critical step in K-Means is determining the number of clusters, `k`. Rather than arbitrarily selecting `k=n`, we followed a structured evaluation process involving:

```python
sse = []
silhouette_scores = []
k_values = range(2, 11)

X = rfm_segmentation.copy()

# Standardize the features
scaler = StandardScaler()
X_scaled = scaler.fit_transform(X)


for k in k_values:
    kmeans = KMeans(n_clusters=k, n_init=1, random_state=42)
    kmeans.fit(X_scaled)
    sse.append(kmeans.inertia_)           # within-cluster sum of squares
    silhouette_scores.append(silhouette_score(X_scaled, kmeans.labels_))
```


#### a) Elbow Method

From k = 2 to k = 4, there was a steep drop, meaning that adding clusters significantly improves the compactness of the groups. After k = 4 or 5, the reduction in SSE started to level off, forming the “elbow.”

<img width="554" height="412" alt="image" src="https://github.com/user-attachments/assets/e026195f-bbfa-4623-b723-a3efc865b286" />

**Insight**: This suggested that around 4–5 clusters strike a good balance between model complexity and explanatory power.


#### b) Silhouette Score

We also computed the **Silhouette Score**, which measures how well each observation lies within its cluster. Higher scores indicate better cohesion and separation.

<img width="569" height="412" alt="image" src="https://github.com/user-attachments/assets/dfe2ecdf-68e2-4bfa-9f9a-f79b7f3e0fbf" />


**Insight**: Scores steadily increased until k = 4 and 5, where the maximum (~0.62) was reached, corroborating the elbow method.



### 3.5 Comparing K Values

To validate the optimal number of clusters, we compared the customer profiles generated with **k=4** and **k=5**.

```python
#fitting data in Kmeans theorem.
kmeans4 = KMeans(n_clusters=4, random_state=0).fit(rfm_segmentation)

# this creates a new column called cluster which has cluster number for each row respectively.
rfm_segmentation['cluster4'] = kmeans4.labels_

print(rfm_segmentation.groupby('cluster4')[['Recency','Frequency','Monetary']].mean())

kmeans5 = KMeans(n_clusters=5, random_state=0).fit(rfm_segmentation)

rfm_segmentation['cluster5'] = kmeans5.labels_

print(rfm_segmentation.groupby('cluster5')[['Recency','Frequency','Monetary']].mean())
```

<img width="311" height="108" alt="image" src="https://github.com/user-attachments/assets/0414a5c6-3e5c-4672-8c2c-e0c0eefc2d74" />
<img width="315" height="127" alt="image" src="https://github.com/user-attachments/assets/34ea17cd-09d1-4f46-b5ce-bba8d8310a5e" />


**k = 4 clusters**

- The segmentation is simple and highly interpretable.  
- It clearly distinguishes:  
  - 🟥 **Cluster 0:** Inactive / low-value customers (high Recency, very low Frequency & Monetary)  
  - 🟦 **Cluster 1:** High-frequency, high-value customers  
  - 🟨 **Cluster 2:** Mid-value loyal customers  
  - 🟩 **Cluster 3:** VIP customers (very recent, very frequent, very high spenders)  
- This structure provides straightforward business insights and is easy to communicate.

**k = 5 clusters**

- Produces a finer level of segmentation.  
- In addition to the four groups above, it introduces a new cluster:  
  - 🟪 **Cluster 4:** New or moderate buyers (recent purchases, but relatively low Frequency & Monetary)  
- This split allows us to differentiate between “emerging customers” and “established mid-value customers,” which can support more personalized marketing strategies.


✅ **Summary**  
- **k=4** → clean and simple segmentation for executive reporting.  
- **k=5** → adds a new segment (**🟪 New / moderate buyers**), allowing for more personalized strategies such as onboarding campaigns or upselling opportunities.  
- Both are valid, and the choice depends on whether the goal is **broad segmentation** or **fine-grained personalization**.  

---

## 4. Insights & Recommendations

This project combines a **rule-based 8-segment RFM model** with a **data-driven K-Means clustering** approach (optimal **k ≈ 4–5** via elbow + silhouette). Together, they provide both **executive-friendly interpretability** and **actionable segmentation** for lifecycle marketing, retention, and monetization.


### 4.1 Key Findings

- **Dual-track segmentation adds value**:  
  - The **8-segment RFM** (binary thresholds on R/F/M) is clear and explainable for stakeholders.  
  - **K-Means** discovers natural groupings; **k=4 or 5** balances compactness, separation, and operational simplicity.
- **Segment granularity trade-off**:  
  - **k=4** → crisp, board-level segmentation (Inactive / Mid-value / High-value / VIP).  
  - **k=5** → adds a **New/Moderate buyers** cohort, enabling finer onboarding & upsell strategies.


### 4.2 Business Playbook (by Segment)
| Segment (typical) | What it means | High-impact actions |
|---|---|---|
| **VIP / High-value** | Recent, frequent, high spend | VIP perks, early access, referral incentives, churn-risk alerts on R dips |
| **High-freq / Mid-value** | Loyal but not premium spend | Cross-sell bundles, AOV boosters, subscription nudges |
| **New / Moderate buyers** *(k=5)* | Recent purchases, moderate spend | Onboarding flows, first-90-day offers, habit-forming campaigns |
| **Mid-value** | Stable recency & frequency | Personalized recommendations, category expansion |
| **Inactive / Low-value** | Long recency, low freq & spend | Win-back promos, re-permissioning, deliverability checks |

> Tip: Keep **RFM labels** (e.g., R=1–4, F=1–4, M=1–4) alongside **K-Means cluster IDs** so marketing can create precise rules (e.g., “Cluster=Mid-value AND R bucket worsened”).


### 4.3 Data Product Implications
- **Feature store & scoring**: nightly R/F/M refresh → scale → **K-Means++ inference** → write cluster IDs + RFM buckets to user profile.  
- **Explainability layer**: expose RFM buckets and simple heuristics for “why this user is VIP vs. Mid-value.”  
- **Monitoring**: track cluster size drift, silhouette at k*, and business KPIs (AOV, CR, LTV) per segment; alert on anomalies.  
- **Outlier handling**: optionally run **DBSCAN** on scaled features to flag extreme spenders or suspicious activity as **noise** for separate review.


### 4.4 Final Recommendation
- **Ship K-Means++ at k=4 or k=5** (choose based on team capacity for distinct playbooks):  
  - If you need **clarity & speed to execute** → **k=4**.  
  - If you need **fine-grained lifecycle programs** → **k=5** (keep the “New/Moderate buyers” cohort).  
- **Retain the 8-segment RFM** as a **transparent baseline & naming schema** to aid communication, QA, and rule-based campaigns.
  

### 4.5 Next Steps
- **Model tuning**: test `log1p` on heavy-tailed Frequency/Monetary; validate **k=4 vs k=5** with business KPIs, not just silhouette.  
- **Experimentation**: A/B test win-back, onboarding, and VIP offers **per segment**; measure uplift in retention and LTV.  
- **Lifecycle automation**: wire clusters into CRM/CDP to trigger journeys (onboarding, cross-sell, reactivation) with guardrails from RFM buckets.  
- **Governance & drift**: monthly review of segment definitions, KPI dashboards, and re-training cadence as behavior shifts.
