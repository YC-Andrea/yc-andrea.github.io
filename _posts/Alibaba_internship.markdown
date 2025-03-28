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
![](/img/36600_poster.pdf)


## Methodology & Tool Stack

### 1. K-means Clustering: Uncovering Hidden Product Patterns
#### Methodology Optimization:
**1. Feature Engineering:**
Selected 6 core features: Price, Total Sales, Reviews, MTD, Estimated Monthly Sales, June Sales  
Applied hybrid normalization:
```ts
from sklearn.preprocessing import StandardScaler, RobustScaler  
# RobustScaler for price/MTD (outlier-prone)  
# StandardScaler for sales/reviews (normal distribution)  
```
**2. Dynamic K-value Selection:**  
Evaluated K=2 to K=10 using multi-criteria:
```ts
from sklearn.metrics import silhouette_score, calinski_harabasz_score  
def optimal_k_search(data, max_k=10):  
    results = []  
    for k in range(2, max_k+1):  
        kmeans = KMeans(n_clusters=k, n_init='auto')  
        labels = kmeans.fit_predict(data)  
        results.append({  
            'K':k,   
            'Inertia':kmeans.inertia_,  
            'Silhouette':silhouette_score(data, labels),  
            'CH_Score':calinski_harabasz_score(data, labels)  
        })  
    return pd.DataFrame(results)  
```
**Breakthrough Findings:**  
Original K=2 analysis split data into outliers vs. normal clusters, missing nuanced patterns.  
K=5 emerged as optimal through multi-dimensional validation:
![](/img/k-means.png）

## 2. Boston Matrix: The Golden Compass for Product Classification
Using market growth rate (Y-axis) and market share (X-axis), 
products were categorized into four quadrants:  

**Stars:** High growth + High share (e.g., ZAM-Recommended Normal SKU 12)  
**Cash Cows:** Low growth + High share (e.g., Normal SKU 4)  
**Question Marks:** High growth + Low share (e.g., Normal SKU 64)  
**Dogs:** Low growth + Low share (e.g., Normal SKU 31)  
**Key Technical Implementation:**
```ts
# Data standardization and quadrant classification  
def categorize_product(growth_rate, market_share):  
    median_growth = np.median(growth_rates)  
    median_share = np.median(market_shares)  
    if growth_rate > median_growth:  
        return "Star" if market_share > median_share else "Question Mark"  
    else:  
        return "Cash Cow" if market_share > median_share else "Dog"  
```



## Key Findings
#### Variable Importance:
Departure delay (DEP_DELAY) was the most significant predictor  
Taxi-out time (TAXI_OUT) and airline (CARRIER) also had notable impact
#### Model Comparison:
Linear regression and random forest performed best (MSE≈57-73)  
KNN performed worst (MSE=128.47), partly due to its inability to handle categorical variables effectively   
Decision trees tended to overfit, showing poor test performance (MSE=103.458)  
#### BIC vs AIC: 
BIC selected a more parsimonious model (1 variable), AIC selected a more complex model (6 variables). This validated the theory that BIC prevents overfitting while AIC tends to fit data better  

## Visual Analysis
We created several key visualizations to understand the data:  
**Variable distribution histograms:** Showed most flight delays concentrated in 0-30 minutes  
**Correlation matrix:** Revealed high correlation between ARR_DELAY and DEP_DELAY (r≈0.9)  
**BIC curve plot:** Helped determine optimal model complexity  
```ts
# BIC curve plotting code
ggplot(data=df.bic, mapping=aes(x=p,y=BIC)) + 
  geom_point(size=1.5,color="blue") + 
  geom_line(color="blue") +
  ylim(min(bic),min(bic+100))
```

## Challenges and Solutions
**Categorical Variable Handling:**  
Random forest limited factors to 53 levels per column, forcing removal of destination variable  
Solution: Consider encoding destinations as geographic regions rather than specific airports  
**Fair Model Comparison:**  
Ensured all models used the same variable set for fair comparison  
Linear regression remained robust even after removing destination variable  
**Computational Efficiency:**  
Random forest and XGBoost required longer training times  
Solution: Used subsampling and parallel computing to accelerate training  


## Lessons Learned

Our analysis demonstrated that simple linear regression performed exceptionally well in predicting flight delays, achieving an adjusted R² of 0.971.  
This project not only deepened my understanding of statistical modeling but also showcased R's powerful capabilities in data science projects—from data cleaning to advanced modeling and result visualization, the R ecosystem provides a complete solution.

