# Enhancing Recipe Quality Prediction through Nutritional Feature Engineering

by Edwin Ruiz & Alexa Fernandez Tobias


---

## Introduction

As people have become more health concsious over the years, the protein content in the foods we consume has become a popular topic of discussion. Those that are interested in going to the gym and overall fitness seem to be particularly concerned over their protein goals and often eat more nutritionally dense, but ultimately less satisfactory meals! On the other hand, the average person not overly-critical of the foods they consume tend to gravitate to lower-protein meals that (objectively) taste better. We wonder if the ratings of high-protein meals receive lower average ratings because of how they may be associated with nourishing and nutritious, though less appetizing, meals.

**Dataset Overview** 

We used two datasets from Food.com:  
- **RAW_recipes.csv:** contains recipe details (name, preparation time, nutrition information, etc.). This dataset contains 93782 rows with 10 columns.

| Column             | Description                                                                                                                                                                                       |
| :----------------- | :------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `'name'`           | Recipe name                                                                                                                                                                                       |
| `'id'`             | Recipe ID                                                                                                                                                                                         |
| `'minutes'`        | Minutes to prepare recipe                                                                                                                                                                         |
| `'contributor_id'` | User ID who submitted this recipe                                                                                                                                                                 |
| `'submitted'`      | Date recipe was submitted                                                                                                                                                                         |
| `'tags'`           | Food.com tags for recipe                                                                                                                                                                          |
| `'nutrition'`      | Nutrition information in the form [calories (#), total fat (PDV), sugar (PDV), sodium (PDV), protein (PDV), saturated fat (PDV), carbohydrates (PDV)]; PDV stands for “percentage of daily value” |
| `'n_steps'`        | Number of steps in recipe                                                                                                                                                                         |
| `'steps'`          | Text for recipe steps, in order                                                                                                                                                                   |
| `'description'`    | User-provided description                                                                                                                                                                         |
| `'ingredients'`    | Text for recipe ingredients                                                                                                                                                                       |
| `'n_ingredients'`  | Number of ingredients in recipe                                                                                                                                                                   |


- **interactions.csv:** contains user reviews and ratings for these recipes. This dataset contains 731927 rows with 5 columns. 

| Column        | Description         |
| :------------ | :------------------ |
| `'user_id'`   | User ID             |
| `'recipe_id'` | Recipe ID           |
| `'date'`      | Date of interaction |
| `'rating'`    | Rating given        |
| `'review'`    | Review text         |

**Given the datasets, we are investigating whether the nutritional quality (the protein content), is associated with user ratings.** After merging the datasets by recipe ID, the resulting data includes nutritional details (extracted from a string column), user ratings, and computed average ratings per recipe. One suggestion is to explicitly define what “high” and “low” protein means (for example, using the median or mean protein value as a cutoff).

To answer this question, we defined “high” and “low” protein recipes:  
- **High-Protein Recipes:** those with protein values greater than or equal to the median protein content  
- **Low-Protein Recipes:** those with protein values below the median

---

## Data Cleaning and Exploratory Data Analysis

In order to properly assess our interests we completed the following data cleaning steps:

1. Left merge the datasets with recipes on id and interactions on recipe_id.
    - This allows us to see each rating tied with the approriate recipe

2. Convert the submitted and date columns to datetime datatype (preference).
3. Replace ratings of 0 with NaN. 
    - Ratings of 0 indicate that someone did not rate the food. In order to avoid a drop in mean ratings, we substitute 0 with NaN which avoids all 0's in future calculations. 

4. Split the nutrition column into sub-features.
    - Orignally, this column is a string structured like a list. We convert each nutritional value in the String to a float in order to be able to perform further analysis. 

5. Concat new float columns created from the nutrition (calories, total_fat, sugar, sodium, protein, saturated_fat, and carbohydrates) column to our merged dataframe.

6. Grouping by recipe id, we computed the average rating per recipe using our merged data. We then merged the resulting dataframe with the original merged_df to add 'average_ratings' as a new column.

Here are the first 5 (unique) rows of our cleaned dataset:


| name                                 | minutes | nutrition                                    | description                                          | rating | calories | protein |
|:-------------------------------------|--------:|:---------------------------------------------|:-----------------------------------------------------|-------:|---------:|--------:|
| 1 brownies in the world best ever    |      40 | [138.4, 10.0, 50.0, 3.0, 3.0, 19.0, 6.0]    | These are the most chocolatey, moist, rich brownies  |    4.0 |    138.4 |     3.0 |
| 1 in Canada chocolate chip cookies   |      45 | [595.1, 46.0, 211.0, 22.0, 13.0, 51.0, 26.0] | This is the recipe that we use at my school cafeteria|    5.0 |    595.1 |    13.0 |
| 412 broccoli casserole               |      40 | [194.8, 20.0, 6.0, 32.0, 22.0, 36.0, 3.0]   | Since there are already 411 recipes for broccoli    |    5.0 |    194.8 |    22.0 |
| Millionaire pound cake               |     120 | [878.3, 63.0, 326.0, 13.0, 20.0, 123.0, 39.0]| Why a millionaire pound cake? Because it's super rich!|    5.0 |    878.3 |    20.0 |
| 2000 meatloaf                        |      90 | [267.0, 30.0, 12.0, 12.0, 29.0, 48.0, 2.0]  | Ready, set, cook! Special edition contest entry      |    5.0 |    267.0 |    29.0 |



**Distribution of Recipe Protein Content:** 

We implemented this plot to get a broader idea of the protein distribution in our merged dataframe. We can see that it is heavily right skewed with only a few recipes going outside of the largest grouped protein. 

<iframe
  src="assets/protein_distribution.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

**Distribution of Average Recipe Ratings:**

We implemented this plot to get a broader idea of the average rating per recipe. This plot is heavily left skewed with the mode as 5 stars. We can see people tend to rate recipes generously with very few below 3 stars and the majority around the 4-5 range. 

<iframe
  src="assets/average_rating_distribution.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

**Protein vs. Average Rating Scatter Plot:** 

Considering our previous interests in protein content and average ratings, we wanted to further investigate any correlations with a scatter plot. As you can see below, as protein content increases, average rating decreases when looking at the trendline (meaning we found a negative correlation between the two variables). Though, a majority of the data is from very low protein content that may be skewing our interpretations. This is why we plan on conducting further analysis. 

<iframe
  src="assets/protein_vs_average_rating.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

Pivot Table: Mean Average Rating by Protein Group & Preparation Time (bin):
| protein_group   |   short |   medium |    long |
|:----------------|--------:|---------:|--------:|
| high            | 4.68556 |  4.67607 | 4.64813 |
| low             | 4.7073  |  4.65943 | 4.67475 |

---

## Assessment of Missingness

In our dataset the 'rating' column has a large number of missing values (15,036 missing). We gathered that people who have neither an exceptionally great nor exceptionally horrible experience (a neutral experience) with a recipe are less likely to rate a recipe as they have less of motive to do so. In other words, whether a person rates a recipe may depend on their subjective experience, which is unobserved in the dataset. This leads us to believe that the 'rating' might be NMAR. If it were MCAR, the fact that we see so many missing values would purely coincidental with no background information as a motive.

However, to determine whether missingness might instead be MAR, we investigate whther the missingness in 'rating' depends on observed features in the dataset. Specifically, we performed a permutation tests on two different variables:
1. `'protein'` – a feature central to our hypothesis
2. `'minutes'` – a feature (priori) that we expect may not influence the decision to leave a rating

By comparing the distributions of these features between rows where 'rating' is missing and where it is not, we can assess if the missingness is systematically related to them.


> Protein and Ratings

**Null Hypothesis:** The missingness of Rating does not depend on the protein amount in the recipe.

**Alternate Hypothesis:** The missingness of Rating does depend on the protein amount in the recipe.

**Test Statistic:** The absolute difference in mean of the distribution of the group without missing ratings and the distribution of the group with missing ratings. 

**Significance Level:** 0.05

We collected 1000 simulated mean differences between the two distriubutions by running a permutation test. This involved shuffling the missingness of rating 1000 times.

<iframe
  src="assets/permutation_test_protein.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

The **observed statistic** of **2.85** is indicated by the red vertical line on the graph. Since the **p_value** that we found **(0.0)** is < 0.05 (significance level), we **reject the null hypothesis** and accept the alternative. This suggests that `'rating'` is MAR, conditional on `'protein'`.

> Minutes and Rating

**Null Hypothesis:** The missingness of ratings does not depend on the length of cooking time (in minutes). 

**Alternate Hypothesis:** The missingness of ratings does depend on the length of cooking time (in minutes)

**Test Statistic:** The absolute difference of mean in cooking time of the recipe in minutes of the distribution of the group without missing ratings and the distribution of the group with missing ratings.

**Significance Level:** 0.05

Since we were also curious if the missingness of ratings was dependent on total preparation time (our minutes column), we again collected 1000 simulated mean differences between the two distriubutions by running a permutation test.

<iframe
  src="assets/permutation_test_minutes.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

The **observed statistic** of **51.45** is indicated by the red vertical line on the graph. The p-value is above 0.05 (with a permutation test p-value of 0.116) indicating that the missing ratings are not systematically associated with how long a recipe takes to prepare. This means that the missingness of the rating column is not dependent on 'minutes' and is likely not driven by preparation time. 

---

## Hypothesis Testing

For our hypothesis, we are comparing average ratings between high-protein and low-protein recipes, thus we need the average_rating value for each recipe and must only drop rows that have missing values in the average_rating column. This will later assure that our hypothesis test (predictive modeling) are based on recipes with complete-meaningful average_ratings.

Our goal is to see if protein and recipes come from the same population. In order to dive deeper into this relationship, we will perform a one-tailed permutation test with the following information:

- **Null Hypothesis (H₀):** There is no difference in the average user rating (as measured by the average_rating column) between high-protein and low-protein recipes
- **Alternative Hypothesis (H₁):** High-protein recipes receive significantly lower average ratings than low-protein recipes

**Test Statistic:**

We want to test whether high-protein recipes get lower ratings, so we do a one-tailed permutation test to see if the mean rating in the high-protein group is lower than the mean rating in the low-protein group. To do this we use the difference in the means of **average_rating** between the two groups as our test statistic. Our reasoning for doing a difference of means is because we want to be able to highlight any directional findings that would not be captured using other test statistics.

**Method**
- **Observed Difference:** Mean (high-protein ratings) – Mean (low-protein ratings) 
- **Permutation Procedure:** We will randomly shuffle the protein-group labels many times (1,000 permutations) and recalculate the difference each time to build an empirical distribution. 
- **P-value:** We will calculate the p-value as the proportion of permuted differences that are less than or equal to the observed difference (if the observed difference is negative). 

<iframe
  src="assets/permutation_test_overall.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

After performing the permutation test, we reached an observed difference of -0.01699 and a test p-value of 0.0. This leads us to reject the null hypothesis and accept the alternative. 

---

## Framing a Prediction Problem

In this section we plan on predicting the continuous variable average_rating for each recipe using the appropriate and available features we have on hand (and at the time of recipe submission). We will treat this as a regression problem seeing as though average_rating ranges approximately from 1-5. 

Our selected features are:
- nutritional values: calories, total_fat, sodium, protein, saturated_fat, and carbohydrates. 
    From our previous analysis we found that protein is correlated with user ratings which suggests other nutrtional factors may also play a role. 
- recipe properties: minutes, n_steps, and n_ingriedients.
    These features may influence how people may perceive or rate a recipe (e.g., ease of preparation, gathering of ingridients)
Our reasoning for not selecting columns such as rating or review is because that is not information we would not have on hand if a rating was yet to be made. All the features we selected is information we would have whether or not someone has already left a rating. 

**Evaluation Metrics**

We used the following metrics to measure our progress:
   - **RMSE (Root Mean Squared Error):** Emphasizes large errors by squaring them before averaging, thus giving a sense of how off the model can be in “worst-case” scenarios  
   - **MAE (Mean Absolute Error):** Straightforward measure of average absolute deviation from true ratings  
   - **R² (Coefficient of Determination):** Indicates how much of the variance in average ratings is explained by the features

Though our main metric is **RMSE.** Our reasoning is because large errors are penalized more than they would be with just MAE. MAE treats all errors equally, but because we're dealing with ratings that implies that an error of being off by an average of <0.4 is the same as an error of being off by an average of 3, which is clearly not the case.

**Why This Matters**

Predicting **average_rating** can guide recipe creators to design or modify recipes that are more likely to garner higher user satisfaction. It also builds upon the earlier hypothesis test that showed a slight negative relationship between high protein and ratings.

Moving forward, we wanted to visualize correlations between the features and the target using a correlation matrix. 

<iframe
  src="assets/correlation_matrix.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

From the correlation matrix, we see that many of the nutritional features (e.g., calories, total_fat, sugar, and carbohydrates) are highly correlated with one another, whereas average_rating does not show a strong linear correlation with any single feature. This entails that a straightforward linear model might not be sufficient to capture how these recipe attributes affect user ratings, which suggests that we need more advanced modeling and feature engineering.

---

## Baseline Model

Baseline Approach:
- Model: Linear Regression
- Data Processing: Appeal **StandardScaler** to all numeric features.
- Data Split: 80% training, 20% test, ensuring we can asses generalization

For our features, we decided to use all nutritional values and recipe properties previously mentioned. These all consist of quantitative features: 

- **Continuous Quantitative:** calories, total_fat, sugar, sodium, protein, saturated_fat, carbohydrates, minutes. 

- **Discrete Quantitative:** n_steps, n_ingredients

Seeing as though none of the features included in our model are categorical (and therefore none are ordinal or nominal), we did not have to perform any encodings.

**Baseline Performance Results**  
- **RMSE:** ~0.4898  
- **MAE:** ~0.3398  
- **R²:** ~0.0006  

<iframe
  src="assets/baseline_pred_vs_actual.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>


Since ratings range from 1 to 5, an RMSE of ~0.4898 indicates that on average we’re off by about half a rating point, but it might still be substantial on a 1–5 scale if we aim for high accuracy. An R² near zero (0.0006) means the baseline model-linear regression with these features (in their current form) doesn’t capture much of the underlying complexity in how users rate recipes. This might be due to non-linear interactions between nutritional content and user preferences that linear regression cannot model well. In conclusion, this baseline model serves as a benchmark; we should explore more sophisticated approaches since there is definitely room for improvement.

---

## Final Model

To improve this baseline, we added two new engineered features (since we've utilized all previous viable features):
- **protein_ratio** ratio of protein to calories (captures nutritional density)
- **sugar_to_carb_ratio:** ratio of sugar to carbohydrates (captures relative sweetness profile)

We did this to capture "nutritional quality" as this is the basis of our research and interest. We created protein_ratio because even though we already have individual measurements for protein and calories, combining them into a single ratio allows us to capture the protein denisty of a recipe. A higher protein ratio indicates that a recipe delivers more protein per calorie (which is often seen as a mark of higher nutritional quality and may positively influence ratings). Additionally, we also developed sugar_to_carb_ratio, which is the proportion of total carbohydrates that come from sugar which helps assess the quality of carbohydrate content. In nutritional terms, not all carbohydrates are equal, a meal with a high sugar_to_carb_ratio suggests that a larger portion of its carbs consists of simple sugars rather than complex carbs. Since a high simple sugar ratio can be percieved as 'unhealthy', we believe they may negatively impact ratings and thus our reasoning for including it in our final model. 

We then used a **RandomForestRegressor** - a non-linear ensemble method that can capture complex interactions. User ratings may depend on interactions between ingredients instead of their independent effects. For example, people sometimes have different perspectives when it comes to the nutritional content depending on their goals so a high protein meal might be positive for a high-caloric meal, but negative for a low-caloric meal. A non-linear model like RandomForest can highlight these interactions, though Linear Regression is unable to do so.

We tuned key hyperparameters using **GridSearchCV** and evaluation is done using the same metrics as above. We tested values for n_estimators (50, 100) and max_depth (None, 10, 20) using GridSearchCV with 5-fold cross-validation, selecting the best model based on RMSE

**Final model training set size:** 185321 rows

**Final model test set size:** 46331 rows

**Best hyperparameters:** {'model__max_depth': None, 'model__n_estimators': 100}

Final Model Performance:

RMSE: 0.3400

MAE: 0.1442

R²: 0.5183

<iframe
  src="assets/final_pred_vs_actual.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

Compared to our baseline model, we see a dramatic difference in accuracy with predicting ratings. 

| **Model**                         | **Features Used**                                                                                           | **Method/Transformations**                                                                                         | **RMSE** | **MAE**  | **R²**   |
|-----------------------------------|-------------------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------|----------|----------|----------|
| **Baseline Model**                      | calories, total_fat, sugar, sodium, protein, saturated_fat, carbohydrates, minutes, n_steps, n_ingredients    | StandardScaler, Linear Regression                                                                                  | 0.4898   | 0.3398   | 0.0006   |
| **Final Model (RandomForest)**    | Baseline features + protein_ratio, sugar_to_carb_ratio                                                      | StandardScaler, RandomForestRegressor (max_depth: None, n_estimators: 100) with GridSearchCV                         | 0.3400   | 0.1442   | 0.5183   |
| **Stacking Ensemble (Post-Final)** | Baseline features + protein_ratio, sugar_to_carb_ratio                                                      | StandardScaler, StackingRegressor with base models: RandomForestRegressor & GradientBoostingRegressor; meta: Ridge   | 0.3384   | 0.1494   | 0.5228   |

<iframe
  src="assets/residual_plots.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

Our analysis set out to determine whether nutritional quality, specifically protein content, is associated with user ratings on Food.com. We began by comparing high‐protein and low‐protein recipes using a permutation test, which rejected the null hypothesis (p-value = 0.0) where we were able to see a trend of high-protein receiving slightly lower ratings. Although the absolute difference in ratings was modest, this finding provided the rational for further predictive modeling. We first began by building a baseline Linear Regression model using 10 core nutritional and recipe features that yielded an RMSE of 0.4898, an MAE of 0.2298, and an R² near zero. This implied that a simple linear approach could not capture the complex factors influencing user ratings. 

We then enhanced our feature set by engineering two new variables—protein_ratio (protein/calories) and sugar_to_carb_ratio (sugar/carbohydrates), and applied a RandomForestRegressor, which substantially improved performance (RMSE: 0.3400, MAE: 0.1442, R²: 0.5183). To explore further enhancements, we implemented a stacking ensemble that combined predictions from RandomForest and Gradient Boosting regressors with a Ridge regressor as the meta-model. The stacking ensemble achieved a slightly better RMSE of 0.3384 and an R² of 0.5228, though its MAE was marginally higher (0.1494). In practical terms, the difference between the RandomForest model and the stacking ensemble is very small—only about a 0.0016 improvement in RMSE and a 0.0045 increase in R². This indicates that both advanced models perform almost equivalently. 

Overall, our results strongly suggest that protein content is a significant factor associated with user ratings; the incorporation of protein-derived features into non-linear models improves predictive accuracy considerably, explaining over 50% of the variance in ratings. While the stacking ensemble offers a minor edge, the increased complexity may not be justified if simplicity and interpretability are prioritized.

---

## Fairness Analysis

To assess if our final RandomForest model performs “fairly” across the attributes we are testing the variable: **Preparation Time** (short vs. long). We chose this variable because while most of our analysis was focused on nutritional value, one's rating may be largely influenced by how long it takes to prepare. Longer recipes may be more intricate and therefore more enjoyable than the average recipe.

We computed the RMSE and performed a **permutation test** to see if any observed difference in RMSE is statistically significant. We used the difference in RMSE as our test statistic and set our significance level at $\alpha = 0.05$. Our reasoning for using RMSE is because of how it measures the average prediction error in the same units as the target variable allowing us to highlight any potential bias.

1. **Grouping Criterion**  
   - We split our test set into “short” vs. “long” preparation time based on the median `minutes` in the final test set

   - **Short Group:** `minutes < median`  
   - **Long Group:** `minutes >= median`

2. **Hypothesis**
   - **Null Hypothesis:** Our model is fair. It's performance is the same for recipes with short preparation time and recipes with long preparation time. Any differences are due to random chance.
   - **Alternative Hypothesis:** Our model is unfair. It's performance differs between the two groups, in particular, recipes with long preparation time have a higher RMSE than those with short preparation time.
3. **Test Statistic**
   - RMSE:
      Difference = RMSE(high) - RMSE(low)
   - Observed difference: 
       RMSE(high) = 0.3355 vs. RMSE(low) = 0.3446 
       low - high = 0.0091
4. **Significance Level**  
   - alpha = 0.05$

5. **Results**
   - observed difference in RMSE: 0.0347
   - p-value: 0.0000

<iframe
  src="assets/fairness_prep_time.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

Since the p-value is extremely small (<0.05), we reject the null hypothesis. The model does appear to have a statistically significant difference in RMSE, performing slightly better for short-prep recipes than long-prep ones.

This suggests that users might see slightly more accurate predictions for recipes with short preparation times. Depending on the context, this disparity might be acceptable, or would warrant further model adjustments and additional features to improve fairness. 
