# Customer-Segmentation-with-RFM-and-Clustering
End-to-end unsupervised ML pipeline segmenting 4,338 retail customers via RFM feature engineering and K-Means clustering. Compares K-Means, Hierarchical, and DBSCAN with silhouette/elbow validation, then translates clusters into actionable marketing segments. Uncovers a Pareto split: 16% of customers drive 65% of revenue.
# Customer Segmentation with RFM and Clustering

End-to-end unsupervised learning pipeline that turns raw retail transactions into
actionable marketing segments.

**Dataset:** [UCI Online Retail](https://archive.ics.uci.edu/dataset/352/online+retail)
— 541,909 transactions, UK-based online retailer (Dec 2010 – Dec 2011).

---

## Key Result

**16% of customers generate 65% of revenue.** Textbook Pareto — and it dictates
exactly how the marketing budget should be allocated.

| Segment | Customers | Revenue Share | Value Index* |
|---|---|---|---|
| 🟢 **Champions** | 713 (16%) | **64.9%** | **3.96** |
| 🔵 **Loyal** | 1,166 (27%) | 23.6% | 0.88 |
| 🟠 **New / Promising** | 837 (19%) | 5.2% | 0.27 |
| 🔴 **At-Risk / Lost** | 1,622 (37%) | 6.2% | 0.17 |

\* Value Index = revenue share ÷ customer share. Above 1.0 means the segment
punches above its weight.

The largest segment (At-Risk, 37%) contributes the least revenue (6%). Spending
heavily to win them back is an ROI mistake.

---

## Pipeline

### 1. Data Cleaning
- Dropped 135K rows with missing `CustomerID` (25% of data — unavoidable for
  customer-level analysis)
- Removed duplicates and cancelled orders (`InvoiceNo` starting with `C`)
- **Kept returns separately** — they feed the `ReturnRate` behavioral feature
  rather than being discarded

### 2. RFM Feature Engineering
Aggregated transaction-level data to customer level:

| Feature | Definition |
|---|---|
| **Recency** | Days since last purchase, relative to a snapshot date (`max(InvoiceDate) + 1 day`) |
| **Frequency** | Number of **distinct invoices** (`nunique`, not `count` — a 20-item basket is one order, not twenty) |
| **Monetary** | Total spend (`Quantity × UnitPrice`) |

Extended with three behavioral features: `AvgOrderValue`, `CategoryDiversity`,
and `ReturnRate`.

### 3. Feature Preparation
Two distinct problems, two distinct fixes:

- **Skew** — `Monetary` skew was **19.3**. K-Means minimizes Euclidean distance,
  so a single £280K customer would hijack an entire cluster. `log1p` transform
  brought skew down to **0.40**.
- **Scale** — `Monetary` spans a range 750× wider than `Recency`. Without
  `StandardScaler`, Monetary alone would dominate every distance calculation.

Order matters: log first, then scale.

### 4. Clustering & Comparison

| Method | Silhouette | Verdict |
|---|---|---|
| **K-Means (k=4)** | **0.3375** | ✅ **Selected** |
| Hierarchical (Ward) | 0.280 | Agrees structurally (ARI = 0.437), but doesn't scale |
| DBSCAN | 0.294 (only 2 clusters) | Structurally unsuited — see below |

**DBSCAN's "failure" is itself a finding.** It searches for density gaps, but RFM
space is a *continuous cloud* — customers grade smoothly from best to worst with
no natural boundaries. The PCA plot confirms this visually. The takeaway: we are
not *discovering* natural clusters, we are *drawing* useful boundaries. That
makes a partitional method the right tool.

### 5. Choosing k
Validated with four metrics, not by eye: **Elbow (inertia), Silhouette,
Davies-Bouldin, Calinski-Harabasz** — plus a dendrogram cut.

**k=2 actually scores higher on silhouette (0.433).** It was rejected anyway:
splitting customers into "good" and "bad" is mathematically clean but useless to
a marketing team. **k=4** sits at a local silhouette maximum, is where the elbow
bends, is where Davies-Bouldin drops, is independently confirmed by the
dendrogram, and maps onto segments a business can actually act on.

In unsupervised learning, the metric is evidence — not the verdict. Actionability
is a criterion too.

---

## Repository

---

## Bonus Features

- **Outlier detection** — Isolation Forest **and** DBSCAN noise points, cross-referenced.
  Surfaced 87 anomalous customers averaging 8× the normal purchase frequency and
  26× the spend. These are **wholesale/B2B buyers** — nobody labeled them, the
  model found them. They should be pulled out of retail campaigns entirely and
  handed to an account manager.
- **t-SNE** — non-linear segment visualization alongside PCA (93.9% variance retained in 2D)
- **Interactive dashboard** — Plotly 3D scatter + executive summary panels
- **Extended RFM** — `CategoryDiversity`, `AvgOrderValue`, `ReturnRate`

---

## Marketing Recommendations

| Segment | Action |
|---|---|
| 🟢 **Champions** | VIP program, referrals, early access. **No discounts** — they buy anyway; discounting is forfeited margin. Highest churn-monitoring priority. |
| 🔵 **Loyal** | Upsell and bundles. **Best ROI in the portfolio** — they already trust the brand, so converting them to Champions is 5–7× cheaper than acquiring new customers. |
| 🟠 **New / Promising** | Onboarding sequence, second-purchase incentive. Time-sensitive — the 1→2 order transition is the make-or-break moment. |
| 🔴 **At-Risk / Lost** | One aggressive win-back attempt, then **cap the budget**. Most will never return; redirect the spend to Loyal. |

---

## Tech Stack

`scikit-learn` · `scipy` · `pandas` · `numpy` · `matplotlib` · `seaborn` · `plotly`

## Running It

```bash
pip install pandas numpy scikit-learn scipy matplotlib seaborn plotly openpyxl nbformat
jupyter notebook customer_segmentation_rfm.ipynb
```

Place `Online_Retail.xlsx` in the project root.
