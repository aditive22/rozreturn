# Rozreturn Market Regime Analysis Report

This report analyzes market regimes in the **BNBFDUSD order and trade data** by leveraging advanced feature engineering and clustering techniques. The overall workflow is implemented in `new.ipynb`.

---

## Table of Contents
1.  [Library Installations](#library-installations)
2. [Data Loading and Feature Engineering](#data-loading-and-feature-engineering)
3. [Data Preprocessing and Dimensionality Reduction](#data-preprocessing-and-dimensionality-reduction)
4. [Clustering Algorithms and Evaluation](#clustering-algorithms-and-evaluation)
5. [Regime Characterization and Insights](#regime-characterization-and-insights)
6. [Visualizations](#visualizations)
7. [Conclusion](#conclusion)


---
## Library Installations
Download and unzip the data files in your directory <br>
To run the analysis, ensure the following Python libraries are installed:

```bash
pip install numpy pandas matplotlib seaborn scikit-learn hdbscan
pip install statsmodels scipy plotly
```
---

## Data Loading and Feature Engineering

### Data Sources:
- **Depth data**: `depth20_1000ms`
- **Trade data**: `aggTrade`
- The datasets are merged using a nearest timestamp approach.

### Custom Engineered Features:
#### Basic Liquidity / Depth:
- **Bid-Ask Spread** and **Relative Spread**
- **Mid Price** and **Imbalance** (Level 1, L5, L10) for measuring book imbalances with division protection.

#### Derived Price Metrics:
- **Microprice**: A price weighted by opposite side quantities.
- **Slope Features**: The `weighted_depth_distance` function computes price deviations weighted by volume, allowing calculation of bid/ask slopes and slope ratio.

#### Volatility & Trend Indicators:
- **Log Return**, **Absolute Return**, and rolling standard deviations over 10s, 30s, and 60s windows.
- **RSI calculations** with built-in protection.
- **Simple Moving Averages (SMA)** for trend detection and a custom **Price Trend Indicator**.
- **Autocorrelation function** (with safeguards) used to capture mean-reversion versus trending behavior.

#### Volume and VWAP:
- Volume imbalances, cumulative volumes, and VWAP-based metrics (including shifts and gaps).

---

## Data Preprocessing and Dimensionality Reduction

### Missing Value Imputation:
- **Returns and volatility-based features**: Filled with `0`.
- **Shift/difference features**: Set to `0` in case of missing data.
- **Other features**: Use the median for imputation.

### Standardization and PCA:
- Data is standardized using **z-score normalization**.
- **PCA** is applied to reduce dimensionality while retaining components that explain at least 90% of the variance.

---

## Clustering Algorithms and Evaluation

### Algorithms Used:
1. **K-Means**:
   - Applied with a range of cluster numbers (from 2 to 10) to obtain metrics and determine an optimal value (set as `K = 4` based on plot results).
2. **HDBSCAN**:
   - Run on PCA-reduced data to handle noise points (with a customized evaluation that excludes negative cluster labels if noise is present).
3. **Gaussian Mixture Model (GMM)**:
   - Evaluated alongside K-Means and HDBSCAN.

### Clustering Metrics:
- **Silhouette Score**: Higher scores indicate better-defined clusters.
- **Davies-Bouldin Score**: Lower scores indicate more compact clusters.
- **Calinski-Harabasz Score**: Higher values are better.

The function `evaluate_clustering` (defined in `main.ipynb`) computes these metrics, including a special case when noise is detected (for HDBSCAN).

### Clustering Results:
- The results of clustering methods are printed and compared.
- The report selects the best method (by default, using KMeans clusters saved under `KMeans_Labels`) and maps these labels back to analyze market regimes further.

---

## Regime Characterization and Insights

### Characterization Function:
The function `characterize_regimes` systematically profiles the clusters by computing:
- **Trend Score**: Combines autocorrelation and absolute price trend indicator.
- **Volatility Score**: Based on short-term volatility and absolute returns.
- **Liquidity Score**: Derived from bid/ask spread and volume metrics.

These scores are combined to assign each regime a name (e.g., “Trending & Liquid & Stable”) and to derive additional statistics such as mean volatility, order book imbalance, and other book-level parameters.

### Regime Insights:
#### Duration Analysis:
- The report computes the transition probabilities between regimes along with the duration each regime persists.

#### Transition Matrix:
- A heatmap visualizes the probability of transitioning from one regime to another.

---

## Visualizations

### Clustering Quality and Metrics:
- **Elbow plot (SSE)** and plots for silhouette, Davies-Bouldin, and Calinski-Harabasz scores.

### Market Regime Evolution:
- Scatter plots display market regimes over time using the mid-price.

### Feature Distributions by Regime:
- Boxplots visualize key features (volatility, bid-ask spread, autocorrelation, order book imbalance) across regimes.

### t-SNE Visualization:
- A t-SNE scatter plot provides a two-dimensional view of the clustering of market regimes.

### Regime Transition and Duration Analysis:
- A heatmap shows transition probabilities.
- Bar plots display average regime durations alongside regime names.

### Feature Importance:
- The report uses `SelectKBest` with the F-test to rank engineered features by their discriminative power across regimes.
- Visuals separately plot the top overall features and breakdowns by categories: trend, volatility, and liquidity.

---

## Conclusion

This report integrates custom feature engineering with multiple clustering techniques to provide deep insights into market regimes. The detailed regime characterizations, combined with rich visualizations, aim to help understand the underlying market structure and the transitions between different regimes.

