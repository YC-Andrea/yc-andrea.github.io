---
layout: post
title: "Alibaba Internship: From Data to Strategy"
subtitle: "Practical Insights from E-commerce Product Analysis in Python"
date:       2023-08-10 12:00:00
author: "Yingqian Cao"
header-style: text
catalog:      true
tags:
  - Internship
  - Python
  - k-means
  - BCG
  - E-commerce
---

## Project Background

In today's thriving e-commerce industry, optimizing product portfolios and enhancing operational 
efficiency through data analysis is a core challenge for businesses. During a two-month internship, 
I conducted an in-depth analysis of non-specialty products for a leading e-commerce platform using 
the Boston Matrix and K-means clustering, completing a full analytical cycle from data cleaning to 
strategic recommendations. This blog shares this technically rigorous and commercially valuable 
analytical journey.


## Methodology & Tool Stack

### 1. Feature Normalization
**Why Normalize?**  
In e-commerce data:
- Price range: ¥13.9 - ¥169  
- Sales range: 200 - 50,000+  
- Transaction growth: -15% to +68%  
- Without normalization, high-magnitude features dominate distance calculations
```ts
from sklearn.preprocessing import MinMaxScaler, StandardScaler
# Process positive features (price/sales)
price_scaler = MinMaxScaler(feature_range=(0,1)) 
normalized_price = price_scaler.fit_transform(df[['price']])
# Process growth data containing negatives
growth_scaler = StandardScaler()
scaled_growth = growth_scaler.fit_transform(df[['growth_rate']])
```

## 2. Deep Dive into K-means Clustering
### 1. Determining Optimal K Value
**Elbow Method Implementation**
```ts
def find_optimal_k(data, max_k=10):
    distortions = []
    for k in range(1, max_k+1):
        kmeans = KMeans(n_clusters=k, n_init=10)
        kmeans.fit(data)
        distortions.append(kmeans.inertia_)
    # Calculate curvature change
    deltas = np.diff(distortions)
    curvature = np.diff(deltas)
    return np.argmax(curvature) + 2  # Return optimal K
optimal_k = find_optimal_k(scaled_data)  
```
**Silhouette Score Validation**
```ts
from sklearn.metrics import silhouette_score
best_score = -1
for k in range(2, 6):
    kmeans = KMeans(n_clusters=k)
    labels = kmeans.fit_predict(scaled_data)
    score = silhouette_score(scaled_data, labels)
    if score > best_score:
        best_k = k
        best_score = score
```
### 2. Feature Combination Experiments
Finding optimal feature combinations through exhaustive search:
```ts
features = ['price', 'sales', 'growth', 'share']
best_combo = []
best_score = 0
for r in range(2, len(features)+1):
    for combo in itertools.combinations(features, r):
        subset = df[list(combo)]
        # Evaluate clustering quality
        score = evaluate_clustering(subset)
        if score > best_score:
            best_combo = combo
            best_score = score
```
![](/img/alibaba_clustering.png)
Final key feature combination:
`Price × Sales × Transaction Growth`  
The experimental results demonstrate that, regardless of whether the cluster number K=5 or K=8, the silhouette coefficients achieved with StandardScaler preprocessing (K=5: 0.736; K=8: 0.747) are significantly higher than those with MinMaxScaler (K=5: 0.432; K=8: 0.459), indicating that StandardScaler more effectively enhances clustering quality. As K increases, both methods show modest improvements in silhouette coefficients (MinMaxScaler +6.3%, StandardScaler +1.5%), suggesting that appropriately increasing the number of clusters can optimize model performance.  

## Boston Matrix Construction Process
### 1. Dynamic Threshold Setting
```ts
# Adaptive threshold calculation
def dynamic_threshold(data):
    q1 = np.percentile(data, 25)
    q3 = np.percentile(data, 75)
    iqr = q3 - q1
    # Exclude outliers
    upper_bound = q3 + 1.5*iqr
    lower_bound = q1 - 1.5*iqr
    return np.clip(data, lower_bound, upper_bound).mean()
market_share_thresh = dynamic_threshold(cleaned_share)
growth_thresh = dynamic_threshold(cleaned_growth)
```
![](/img/BCG_Matrix.png)
In this analytical report, we applied the Boston Matrix framework to visualize collected data on a two-dimensional chart, with market growth rate plotted on the Y-axis and market share on the X-axis. Each product was then positioned within the matrix according to its individual market share and growth rate metrics. This approach enabled us to conduct focused analysis on products.

### 2. Four-Quadrant Classification Logic
```ts
graph TD
    A[Raw Data] --> B{Market Share ≥2%?}
    B -->|Yes| C{Growth Rate ≥4%?}
    C -->|Yes| D[Star Products]
    C -->|No| E[Cash Cows]
    B -->|No| F{Growth Rate ≥4%?}
    F -->|Yes| G[Question Marks]
    F -->|No| H[Dogs]
```
### 3. Outlier Handling Strategy
Identified three special cases:
```ts
outliers = df[
    (df['growth_rate'] > growth_thresh*3) | 
    (df['market_share'] > share_thresh*3)
]
# Handling approach:
# 1. High-growth outliers: Keep and flag separately
# 2. High-share outliers: Validate with operations team
# 3. Negative-growth anomalies: Classify as special observation group under Dogs
```




## Lessons Learned

This analysis has given me profound insight that data analytics is not just a technical task,
but an extension of business understanding. From data cleaning and normalization to K-means clustering 
and Boston Matrix construction, each step requires finding a balance between mathematical rigor and business 
logic. For example, while the Elbow Method and Silhouette Coefficient for K-means provide theoretical guidance,
the final choice of K-value must consider business context; the dynamic threshold setting for the Boston Matrix 
made me realize there are no "standard answers" in data, only "optimal solutions."  


