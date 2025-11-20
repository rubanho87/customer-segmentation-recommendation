# Customer Segmentation & Recommendation System

## 🎯 Objective

Build, from a retail transaction history, a complete and transparent example of:

- customer segmentation using **RFM (Recency, Frequency, Monetary)**,
- unsupervised clustering with **K-means**,
- simple **product recommendation** based on customer segments.

The goal is to show the full workflow from raw data to actionable insights.

---

## 📊 Data

- **Source**: Kaggle dataset *“Customer Segmentation & Recommendation System”* (online retail transactions).
- **File**: `data/raw_data/data.csv`
- **Main columns**: `InvoiceNo`, `StockCode`, `Description`, `Quantity`, `InvoiceDate`, `UnitPrice`, `CustomerID`, `Country`.

No sensitive or personal data is used.

---

## 🔁 Project pipeline (notebook)

All steps are implemented in the notebook:

`customer_segmentation_recommendation_online_retail.ipynb`

Main sections:

1. **Loading the data**
2. **Cleaning & preparation**
   - remove invalid rows (missing `CustomerID`, negative quantities/prices, cancellations)
   - create `TotalPrice = Quantity × UnitPrice`
   - ensure dates and types are correct
3. **Exploratory analysis**
   - basic KPIs (number of customers, products, countries, total revenue)
   - top products (transactions, quantity, revenue)
   - revenue and active customers per month
4. **RFM construction**
   - Recency = days since last purchase  
   - Frequency = number of invoices  
   - Monetary = total revenue per customer
5. **RFM scoring (1–5) & mini-segmentation**
   - rule-based scores for R, F, M
   - global `RFM_score` and simple labels:
     - `VIP / High value`
     - `Loyal`
     - `Occasional`
     - `At risk / Low value`
6. **K-means clustering**
   - log-transform & scaling of R, F, M
   - choice of number of clusters (elbow + silhouette)
   - cluster profiling (average R, F, M, size)
7. **Top products by cluster**
   - products that generate the highest revenue in each cluster
8. **Simple recommendation system**
   - for a given customer:
     - find their cluster,
     - select the best products in that cluster,
     - remove products already bought,
     - return a ranked list of recommendations.

---

## 📂 Processed datasets

The notebook can export cleaned datasets to `data/processed_data/`:

- `transactions_clean.csv` – cleaned transactions with `TotalPrice` and `YearMonth`
- `transactions_with_clusters.csv` – transactions + customer cluster and label
- `customers_rfm_clusters.csv` – one row per customer with RFM, scores and clusters
- `top_products_by_cluster.csv` – aggregated view of top products per cluster

These files are useful for dashboards (Power BI, Tableau, etc.).

---

## 🛠 Environment

Main libraries:

- `pandas`, `numpy`
- `scikit-learn`
- `matplotlib` (optionally `seaborn`)
- `jupyter`

All dependencies are listed in **`requirements.txt`**.

---

## ▶️ How to run locally

```bash
# 1. Clone the repo
git clone https://github.com/rubanho87/customer-segmentation-recommendation.git
cd customer-segmentation-recommendation

# 2. Create and activate a virtual environment (optional but recommended)

# 3. Install dependencies
pip install -r requirements.txt

# 4. Start Jupyter and open the notebook
jupyter notebook
