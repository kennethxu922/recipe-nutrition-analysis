---
layout: single
title: Optimizing Muscle Retention
---

# Optimizing Muscle Retention: Identifying High-Protein, Low-Calorie Recipes  
**By Kenneth Xu**  
[kennethx@umich.edu] | [Notebook PDF](assets/cut-nut.pdf)  

---

## Introduction  
**Key Question**  
*Which high-protein, low-calorie recipes have the best ratings to help maintain muscle during a calorie deficit?*  

**Dataset Overview**  
- **83782** recipes  
- **731927** user interactions/reviews
- **234429** rows after cleaning and merging
- **Key (Relevant) Features**:  
  - `protein_pdv`: Protein (% Daily Value per serving)  
  - `calories`: Calories per serving  
  - `avg_rating`: Average user rating (1–5 stars)  
  - `minutes`: Preparation time  

**Why This Matters**  
When cutting calories, maintaining muscle mass requires strategic nutrition. This analysis helps fitness enthusiasts find tasty, nutrient-dense recipes that support their goals.

---

## Data Cleaning and Exploratory Data Analysis  

### Data Cleaning  
1. **Merged Data**: Left-merged the recipe metadata (`RAW_recipes.csv`) with 234,429 user interactions (`RAW_interactions.csv`) on `id = recipe_id` to ensure every recipe—rated or unrated—appears in our analysis.  
2. **Handled Missing Values**:   
2. **Handled Missing Values**:  
   - Replaced all `rating == 0` entries (which indicate “no rating”) with `NaN`.  
   - Dropped **2,777** recipes whose aggregated `avg_rating` remained `NaN` (i.e., received no valid ratings).   
3. **Parsed Nutrition Data**: Converted the packed `nutrition` string into numeric columns.  
4. **Computed Metric**: Grouped by `recipe_id` and took the mean of all valid `rating` values to create a single `avg_rating` per recipe.  

**Cleaned Data Sample with Relevant Features**:  

| name                                 |   calories |   protein_pdv |   avg_rating |   minutes |
|:-------------------------------------|-----------:|--------------:|-------------:|----------:|
| 1 brownies in the world    best ever |      138.4 |             3 |            4 |        40 |
| 1 in canada chocolate chip cookies   |      595.1 |            13 |            5 |        45 |
| 412 broccoli casserole               |      194.8 |            22 |            5 |        40 |
| 412 broccoli casserole               |      194.8 |            22 |            5 |        40 |
| 412 broccoli casserole               |      194.8 |            22 |            5 |        40 |


### Univariate Analysis  
**Protein Distribution**  
<iframe src="assets/protein-distribution.html" width=800 height=500 frameborder=0></iframe>  
*The histogram shows that the vast majority of recipes are low-protein – over 80 000 recipes (about 35 %) provide less than 10 % DV and roughly 60 % fall below 20 % DV. In contrast, high-protein recipes (≥40 % DV) are comparatively scarce, making up only a small tail of the distribution which makes our project so meaningful in finding recipes that satisfy taste and caloric/protein goals*

### Bivariate Analysis  
**Calories vs. Ratings**  
<iframe src="assets/calories-vs-rating.html" width=800 height=500 frameborder=0></iframe>  
*This scatter plot shows that recipe ratings cluster tightly between 4.0–5.0 stars regardless of calorie count (0–2000 cal), indicating there’s no systematic trade-off between calories and taste, there's actually many low calorie dishes that taste amazing which is great for us!*

### Interesting Aggregates  
**Protein Tier Summary**  

| protein_pdv         |   Avg Rating |   Median Calories |   Recipe Count |
|:--------------------|-------------:|------------------:|---------------:|
| Very Low (0-20%)    |      4.68035 |            196.8  |         111693 |
| Low (20-30%)        |      4.6731  |            347.5  |          21844 |
| Moderate (30-40%)   |      4.68348 |            374    |          16334 |
| High (40-60%)       |      4.6644  |            412.6  |          28758 |
| Very High (60-100%) |      4.66542 |            567.75 |          29616 |

*By grouping recipes into protein %DV tiers, we see that the High (40–60% DV) category sustains the top average rating (4.7★) while keeping median calories at a moderate 413, making it the ideal balance of taste and nutrition for muscle retention.*

### Imputation  
No additional imputation beyond dropping unrated recipes was necessary, since all remaining columns were complete after cleaning.

---

## Framing a Prediction Problem  

**Key Question**  
*Can we predict a recipe’s average rating (1–5 stars) based solely on its nutritional profile and preparation complexity, to identify high-protein, low-calorie recipes that are likely to taste good?*  

**Task**: Regression (predict continuous `avg_rating`, 1–5 stars)  
We use regression because the target is a continuous score where differences (e.g. 4.2 vs. 4.8) are meaningful.  

**Response Variable**:  
- `avg_rating` (mean user rating, 1–5 stars) — chosen as a direct measure of perceived recipe quality.  

**Features**:  
- `protein_pdv`, `calories` (nutritional profile)  
- `minutes` (prep time)  
*All of these features are known before cooking, eliminating any risk of data leakage.*  

**Evaluation Metric**: R² Score  
*Selected because it quantifies the proportion of variance in the continuous rating explained by our model, aligning with our goal of capturing nuanced differences in recipe quality. In other words, r^2 measures how well the features correlate to the overall quality of a recipe*  

---

## Baseline Model  
**Pipeline**  
*My baseline pipeline uses a simple LinearRegression on two quantitative features—protein_pdv and calories—with no ordinal or nominal variables (so no encodings beyond standardization). Both features are scaled via a StandardScaler before fitting. The model yields an R² of 0.000 on both the training and test sets, meaning it explains essentially none of the variance in avg_rating. Given these results, it’s not a “good” model: protein and calorie alone don’t meaningfully predict recipe quality. The small negative coefficients (–0.005 for protein; –0.003 for calories) further confirm that any apparent relationship is negligible and counterintuitive.*

## Final  Model
*I augmented our feature set with protein_per_calorie, which captures the density of muscle-building nutrients, and is_quick_meal, a binary flag for recipes with ≤30 min prep, because users often prefer nutrient-dense dishes that are also convenient. These engineered features encode real-world preferences—high protein concentration and quick preparation—that we expect to drive higher ratings. I then switched to a RandomForestRegressor and used GridSearchCV (2-fold CV on 10 k samples) to tune n_estimators=50 and max_depth=10, balancing prediction power against training speed. The final model’s test R² of 0.042 improves on the baseline’s R² of 0.000, showing that the non-linear tree ensemble can leverage our new features to explain additional variance in user ratings. Although modest, this uplift confirms that combining protein density with prep-time heuristics better captures what makes a recipe taste good.*

