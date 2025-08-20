# 🚀 Yena's Data Science Portfolio – Real Problems, Scalable Solutions

Hi! I'm a data scientist with a passion for solving **business-critical problems** using data, modeling, and clear storytelling.

I believe that good analysis starts with good questions — and ends with clarity, action, and value.  
This portfolio includes real-world projects in **experimentation**, **recommendation**, **customer analysis**, **NLP**, and **time series forecasting**.  
Each one reflects my ability to turn messy data into insights that matter.

> 🎯 Business-first. Impact-focused. Insight-driven.


> If you're a recruiter, teammate, or collaborator — let's connect!  
> 📎 [Connect with me on LinkedIn](https://www.linkedin.com/in/你的linkedin用户名) · ✉️ [Email Me](mailto:yenawei@yahoo.com)


---

## 🔍 What I Do Best

- **Business Impact First**: Modeling that ties directly to retention, ROI, churn, and growth.
- **End-to-End Delivery**: From raw data to insight to model to deployment.
- **Scalable Thinking**: Modular code, reproducible pipelines, interpretable outputs. 

---

## 🔧 Skills Summary

**Languages**: Python · SQL  
**Core Skills**: EDA · A/B Testing · Time Series Analysis · Recommendation Systems · Customer Segmentation  
**Libraries/Tools**: Pandas · Scikit-learn · XGBoost · ARIMA · KMeans · Seaborn · Matplotlib · TextBlob · NetworkX · Folium

---

## 📂 Portfolio Projects

### 🧭 Customer Segmentation with RFM + KMeans
**Goal.** Build actionable customer segments from transactions to drive retention, upsell, and win-back strategies.

**Method.**
- Feature engineering: RFM (Recency, Frequency, Monetary) → log transform (when needed) → **StandardScaler**
- Modeling: **K-Means++** with high `n_init` and increased `max_iter`; compared **k=2..10**
- Model selection: **Elbow (SSE)** + **Silhouette** (peak around **k = 4–5**)
- Baselines & robustness: **Rule-based 8-segment RFM** for interpretability; **DBSCAN** for outlier sanity-check

**Results.**
- **k=4**: clear, board-level segmentation (Inactive / Mid-value / High-value / VIP)  
- **k=5**: adds **New/Moderate buyers** → better onboarding and early-life campaigns  
- Produced per-cluster R/F/M profiles and example actions (VIP perks, cross-sell bundles, win-back promos)

**Impact.**
- Converts raw transactions into a **deployable segmentation layer** for CRM/CDP
- Balances **explainability (8-segment RFM)** with **data-driven grouping (K-Means++)**
- Ready for A/B tests on retention and LTV uplift per segment
  
🔗 [View Project](https://github.com/Nana47lucky/DS_Portfolio/blob/main/01_Customr_Segmentation_RFM_KMeans/readme.md)

---

