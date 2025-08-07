# 👥 Customer Segmentation with RFM & K-Means Clustering

---

## 1. Project Overview

### 1.1 Context and Objective

Customer segmentation is a foundational step in personalized marketing and data-driven customer relationship management (CRM). By classifying customers based on their transaction history, businesses can optimize resource allocation, enhance retention efforts, and maximize profitability.

In this project, we apply the RFM model—which stands for Recency, Frequency, and Monetary value—to segment customers using transaction data. We further enhance segmentation insights by applying unsupervised clustering algorithms including K-Means and DBSCAN, and visually comparing their performance.

> **Business Value:** Help the marketing team personalize communication, optimize resource allocation, and focus on high-value customers.

---

## 2. Data Overview & Preprocessing

### 2.1 Dataset Summary

The dataset covers transactions between Dec 1, 2010, and Dec 9, 2011, and includes:

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

### 2.2 Cleaning Steps

- Removed cancellations (`InvoiceNo` starts with 'C')
- Dropped null `CustomerID` rows
- Filtered negative/zero quantities or prices
- Added `TotalPrice = Quantity * UnitPrice`
- Defined reference date as **2011-12-10** (1 day after last invoice)

---

## 3. RFM Feature Engineering

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


### 3.2 Calculating Recency

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


---

## 4. Rule-Based Segmentation

### 4.1 Labeling via Thresholds

We normalized each RFM feature and binarized it based on whether it was above or below the mean. This created a 3-bit string (e.g., "011"), which was mapped to a customer label:

```python
def rfm_func(x):
    level = x.apply(lambda x: '1' if x > 0 else '0')
    key = level.Recency + level.Frequency + level.Monetary
    return {
        "011": "High-Value Customer",
        "111": "Loyal Customer",
        "001": "At-Risk High Spender",
        "101": "Potential Loyalist",
        "110": "Value Customer",
        "010": "Average Loyalist",
        "100": "At-Risk Retainer",
        "000": "New or Inactive"
    }[key]
```

---

## 5. Clustering Methods

RFM segmentation provides a powerful way to classify customers based on Recency, Frequency, and Monetary value. However, assigning customers into fixed categories via manual RFM score thresholds can oversimplify the underlying customer behaviors. To uncover **natural groupings** in the data without predefined labels, we implemented **unsupervised clustering algorithms**.

We began with **K-Means clustering**, a widely used technique for partitioning observations into cohesive groups based on feature similarity. The goal was to let the data drive the segmentation, and compare it later with DBSCAN results.


### 5.1 Choosing the Optimal K

A critical step in K-Means is determining the number of clusters, `k`. Rather than arbitrarily selecting `k=3`, we followed a structured evaluation process involving:

```python
import numpy as np
import matplotlib.pyplot as plt
from sklearn.cluster import KMeans
from sklearn.metrics import silhouette_score
from sklearn.preprocessing import StandardScaler

rfm_segmentation = rfm[['Recency','Frequency','Monetary']].copy()

# Determine the optimal value of k using the elbow method
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
    sse.append(kmeans.inertia_)
    silhouette_scores.append(silhouette_score(X_scaled, kmeans.labels_))


# kmeans++
#for k in k_values:
#    kmeans = KMeans(n_clusters=k, init='k-means++', random_state=42)
#    kmeans.fit(X)


# Plot the elbow curve
plt.plot(k_values, sse, 'bo-')
plt.xlabel('Number of Clusters (k)')
plt.ylabel('Sum of Squared Errors (SSE)')
plt.title('Elbow Curve')
plt.show()

# Plot the silhouette scores
plt.plot(k_values, silhouette_scores, 'bo-')
plt.xlabel('Number of Clusters (k)')
plt.ylabel('Silhouette Score')
plt.title('Silhouette Scores')
plt.show()
```


#### a) Elbow Method

We plotted the **Within-Cluster Sum of Squares (WCSS)** against increasing values of `k` (from 1 to 10). The elbow point—where the reduction in WCSS begins to diminish—suggested a good balance between **underfitting** and **overfitting**.

<img width="554" height="412" alt="image" src="https://github.com/user-attachments/assets/e026195f-bbfa-4623-b723-a3efc865b286" />

**Insight**: A noticeable elbow appeared at **k = 3**, where the WCSS dropped sharply and then leveled off.


#### b) Silhouette Score

We also computed the **Silhouette Score**, which measures how well each observation lies within its cluster. Higher scores indicate better cohesion and separation.

<img width="569" height="412" alt="image" src="https://github.com/user-attachments/assets/dfe2ecdf-68e2-4bfa-9f9a-f79b7f3e0fbf" />


**Insight**: The silhouette score peaked around **k = 3**, corroborating the elbow method.



### 5.2 K-Means Clustering Results

With `k=3` determined, we ran K-Means clustering on the scaled RFM features. The resulting clusters revealed distinct behavioral groups.

#### Cluster Distribution

- **Cluster 0**: Low frequency and low monetary users — likely low-value or dormant customers.
- **Cluster 1**: Moderate engagement — stable and average customers.
- **Cluster 2**: High spenders and frequent buyers — valuable customers to retain.



### 5.3 Comparison with DBSCAN

To validate our K-Means segmentation, we compared it with **DBSCAN**, a density-based algorithm that does not require `k` and can identify noise/outliers.

#### DBSCAN Clustering Results

<img width="600" height="610" alt="image" src="https://github.com/user-attachments/assets/71d65de5-5aa8-4da6-83b4-21b6099bd7ae" />


- DBSCAN assigned most customers to a single cluster and flagged many as noise.
- While robust to outliers, DBSCAN failed to capture the nuanced variation in customer behavior due to the density assumptions.

---

## 6. Insights & Recommendation

### 6.1 Key Findings

Through a combination of rule-based RFM segmentation and unsupervised clustering algorithms, this project revealed meaningful customer groupings that support targeted marketing strategy. Key insights include:

- **Rule-based segmentation** using RFM score thresholds generated intuitive personas like *High-Value Customers*, *Loyal Customers*, and *At-Risk Retainers*, which are easy to interpret and actionable for marketing.
- **K-Means clustering (k=3)** provided well-separated clusters corresponding to behavioral patterns across spending, frequency, and recency.
    - Cluster 0: Inactive or low-value customers
    - Cluster 1: Mid-tier, stable customers
    - Cluster 2: High-value customers



### 6.2 Business Impact

The segmentation output supports strategic marketing in the following ways:

- **Personalized Campaigns**: Target *High-Value* and *Loyal* customers with loyalty rewards and VIP perks to reinforce retention.
- **Reactivation Programs**: Engage *At-Risk Retainers* and *Inactive Customers* with special offers to win them back.
- **Customer Lifecycle Marketing**: Map clusters to lifecycle stages and tailor communication frequency and content accordingly.
- **Resource Optimization**: Prioritize marketing spend on segments with the highest potential ROI.

These data-driven insights can directly enhance marketing efficiency, reduce churn, and improve Customer Lifetime Value (CLV).

