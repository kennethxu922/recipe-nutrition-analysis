# Optimizing Muscle Retention: Identifying High-Protein, Low-Calorie Recipes  
**By Kenneth Xu**  
[kennethxu@umich.edu] | [Notebook PDF](link-to-your-pdf)  

---

## Introduction  
**Key Question**  
*Which high-protein, low-calorie recipes have the best ratings to help maintain muscle during a calorie deficit?*  

**Dataset Overview**  
- **231,652** recipes after cleaning  
- **Key Features**:  
  - `protein_pdv`: Protein (% Daily Value per serving)  
  - `calories`: Calories per serving  
  - `avg_rating`: Average user rating (1–5 stars)  
  - `minutes`: Preparation time  

**Why This Matters**  
When cutting calories, maintaining muscle mass requires strategic nutrition. This analysis helps fitness enthusiasts find tasty, nutrient-dense recipes that support their goals.

---

## Data Cleaning and Exploratory Data Analysis  

### Data Cleaning  
1. **Merged Data**: Combined recipe details with 234,429 user interactions.  
2. **Handled Missing Values**:  
   - Replaced 0 ratings with `NaN`.  
   - Dropped 2,777 recipes with no ratings.  
3. **Parsed Nutrition Data**: Converted the packed `nutrition` string into numeric columns.  
4. **Computed Metric**: Calculated each recipe’s `avg_rating`.  

**Cleaned Data Sample**:  
<iframe src="assets/cleaned-data-sample.html" width=800 height=200 frameborder=0></iframe>

### Univariate Analysis  
**Protein Distribution**  
<iframe src="assets/protein-distribution.html" width=800 height=500 frameborder=0></iframe>  
*68% of recipes provide <20% DV protein; focus shifts to the 40–60% DV “High” tier.*

### Bivariate Analysis  
**Calories vs. Ratings**  
<iframe src="assets/calories-vs-rating.html" width=800 height=500 frameborder=0></iframe>  
*No clear correlation (R²≈0) – low-calorie dishes can be highly rated.*

### Interesting Aggregates  
**Protein Tier Summary**  
<iframe src="assets/protein-tiers-table.html" width=700 height=300 frameborder=0></iframe>  
*“High” protein (40–60% DV) recipes sustain 4.7★ ratings at a median 413 calories.*

### Imputation  
No additional imputation beyond dropping unrated recipes was necessary, since all remaining columns were complete after cleaning.

---

## Framing a Prediction Problem  
**Task**: Regression (predict continuous `avg_rating`, 1–5 stars)  
**Features**:  
- `protein_pdv`, `calories` (nutritional profile)  
- `minutes` (prep time)  

**Evaluation Metric**: R² Score  
*Chosen to measure how much variance in user ratings our model explains.*

---

## Baseline Model  
**Pipeline**  
```python
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import StandardScaler
from sklearn.linear_model import LinearRegression

baseline_pipeline = Pipeline([
    ('scaler', StandardScaler()),
    ('regressor', LinearRegression())
])
