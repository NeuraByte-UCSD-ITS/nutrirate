# Enhancing Recipe Quality Prediction through Nutritional Feature Engineering

by Edwin Ruiz & Alexa Fernandez Tobias


---

## Introduction

As people have become more health concsious over the years, the protein content in the foods we consume has become a popular topic of discussion. Those that are extremely interested in going to the gym and overall fitness seem to obssess over their protein goals and often eat more nutritionally dense, but ultimately less satisfactory meals. On the other hand, the average person not overly-critical of the foods they consume tend to gravitate to lower-protein meals that (objectively) taste better. We wonder if the ratings of high-protein meals recieve lower average ratings because of how they may be associated with nourishing and nutritious, though less appetizing, meals.

**Dataset Overview** 
This project uses two datasets from Food.com:  
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

**Given the datasets, we are investigating whether the nutritional quality (the protein content), is associated with user ratings** After merging the datasets by recipe ID, the resulting data includes nutritional details (extracted from a string column), user ratings, and computed average ratings per recipe. One suggestion is to explicitly define what “high” and “low” protein means (for example, using the median or mean protein value as a cutoff).

To answer this question, we defined “high” and “low” protein recipes:  
- **High-Protein Recipes:** those with protein values greater than or equal to the median protein content  
- **Low-Protein Recipes:** those with protein values below the median

**Hypotheses for the Association**  
- **Null Hypothesis (H₀):** There is no difference in the average user rating (as measured by the “average_rating” column) between high-protein and low-protein recipes
- **Alternative Hypothesis (H₁):** High-protein recipes receive significantly lower average ratings than low-protein recipes

**Modeling Context**  
If we later build a model to predict ratings, the target variable will be **average_rating** (a continuous variable), making this a regression problem.

---

## Cleaning and EDA

<iframe src="assets/10-80-enrollment.html" width=800 height=600 frameBorder=0></iframe>

In order to properly assess our interests we completed the following data cleaning steps:

1. Left merge the datasets with recipes on id and interactions on recipe_id
    This allows us to see each rating tied with the approriate recipe

2. Converte the submitted and date columns to datetime datatype

3. Replace ratings of 0 with NaN. 
    Ratings of 0 indicate that someone did not rate the food. In order to avoid a drop in mean ratings, we substitute 0 with NaN which avoids all 0's in future calculations. 

4. Parse the nutrition column
    Orignally, this column is a string structured like a list. We convert each nutritional value in the String to a float in order to be able to perform further analysis. 

5. Concat new float columns created from the nutrition (calories, total_fat, sugar, sodium, protein, saturated_fat, and carbohydrates) column to our merged dataframe.

6. Grouping by recipe id, we computed the average rating per recipe using our merged data. We then merged the resulting dataframe with the original merged_df to add 'avg_ratings' as a new column.

Here are the first 5 (unique) rows of our cleaned dataset:
| name                                 |   minutes | nutrition                                     | description                                                                                                                                                                                                                                                                                                                                                                       |   rating |   calories |   protein |
|:-------------------------------------|----------:|:----------------------------------------------|:----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------:|-----------:|----------:|
| 1 brownies in the world    best ever |        40 | [138.4, 10.0, 50.0, 3.0, 3.0, 19.0, 6.0]      | these are the most; chocolatey, moist, rich, dense, fudgy, delicious brownies that you'll ever make.....sereiously! there's no doubt that these will be your fav brownies ever for you can add things to them or make them plain.....either way they're pure heaven!                                                                                                              |        4 |      138.4 |         3 |
| 1 in canada chocolate chip cookies   |        45 | [595.1, 46.0, 211.0, 22.0, 13.0, 51.0, 26.0]  | this is the recipe that we use at my school cafeteria for chocolate chip cookies. they must be the best chocolate chip cookies i have ever had! if you don't have margarine or don't like it, then just use butter (softened) instead.                                                                                                                                            |        5 |      595.1 |        13 |
| 412 broccoli casserole               |        40 | [194.8, 20.0, 6.0, 32.0, 22.0, 36.0, 3.0]     | since there are already 411 recipes for broccoli casserole posted to "zaar" ,i decided to call this one  #412 broccoli casserole.i don't think there are any like this one in the database. i based this one on the famous "green bean casserole" from campbell's soup. but i think mine is better since i don't like cream of mushroom soup.submitted to "zaar" on may 28th,2008 |        5 |      194.8 |        22 |
| millionaire pound cake               |       120 | [878.3, 63.0, 326.0, 13.0, 20.0, 123.0, 39.0] | why a millionaire pound cake?  because it's super rich!  this scrumptious cake is the pride of an elderly belle from jackson, mississippi.  the recipe comes from "the glory of southern cooking" by james villas.                                                                                                                                                                |        5 |      878.3 |        20 |
| 2000 meatloaf                        |        90 | [267.0, 30.0, 12.0, 12.0, 29.0, 48.0, 2.0]    | ready, set, cook! special edition contest entry: a mediterranean flavor inspired meatloaf dish. featuring: simply potatoes - shredded hash browns, egg, bacon, spinach, red bell pepper, and goat cheese.                                                                                                                                                                         |        5 |      267   |        29 |

| name                                 |     id |   minutes | submitted           |   rating |   avg_rating |   calories  |   sugar   | is_dessert   |   prop_sugar |
|:-------------------------------------|-------:|----------:|:--------------------|---------:|-----------------:|---------------:|--------------:|:-------------|-------------:|
| 1 brownies in the world    best ever | 333281 |        40 | 2008-10-27 00:00:00 |        4 |                4 |          138.4 |            50 | True         |    0.361272  |
| 1 in canada chocolate chip cookies   | 453467 |        45 | 2011-04-11 00:00:00 |        5 |                5 |          595.1 |           211 | False        |    0.354562  |
| 412 broccoli casserole               | 306168 |        40 | 2008-05-30 00:00:00 |        5 |                5 |          194.8 |             6 | False        |    0.0308008 |
| millionaire pound cake               | 286009 |       120 | 2008-02-12 00:00:00 |        5 |                5 |          878.3 |           326 | True         |    0.371172  |
| 2000 meatloaf                        | 475785 |        90 | 2012-03-06 00:00:00 |        5 |                5 |          267   |            12 | False        |    0.0449438 |


**Distribution of Recipe Protein Content** 
We implemented this plot to get a broader idea of the protein distribution in our merged dataframe. We can see that it is heavily right skewed with only a few recipes going outside of the largest grouped protein. 

<iframe
  src="assets/protein_distribution.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

**Distribution of Average Recipe Ratings**
We implemented this plot to get a broader idea of the average rating per recipe. This plot is heavily left skewed with the mode as 5 stars. We can see people tend to rate recipes generously with very few below 3 stars and a majority around the 4-5 range. 

<iframe
  src="assets/average_rating_distribution.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

**Protein vs. Average Rating Scatter Plot** 
Considering our previous interests in protein content and average ratings, we wanted to further investigate any correlations with a scatter plot. As you can see below, as protein content increases, average rating decreases when looking at the trendline. Though, a majority of the data is from very low protein content that may be skewing our interpretations which is why we plan on conducting further analysis. 

<iframe
  src="assets/protein_vs_average_rating.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

---

## Assessment of Missingness

- In our dataset the 'rating' column has a large number of missing values (15,036 missing). We gathered that people who have neither an exceptionally great nor exceptionally horrible experience with a recipe are less likely to rate a recipe as they have less incentives to do so. This leads us to believe that this column is NMAR 

We now investigate whether the missingness of the 'rating' column depends on other features by performing permutation tests on two different variables:
1. `'protein'` – a feature central to our hypothesis
2. `'minutes'` – a feature (priori) that we expect may not influence the decision to leave a rating

By comparing the distributions of these features between rows where 'rating' is missing and where it is not, we can assess if the missingness is systematically related to them.

> Protein and Ratings

**Null Hypothesis:** The missingness of Rating does not depend on the protein amount in the recipe.

**Alternate Hypothesis:** The missingness of Rating does depend on the protein amount in the recipe.

**Test Statistic:** The absolute difference of mean in of the recipe of the distribution of the group without missing ratings and the distribution of the group with missing descriptions. 

**Significance Level:** 0.05

<iframe
  src="assets/distr_rating_sugar.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

We ran a permutation test by shuffling the missingness of rating for 1000 times to collect 1000 simulating mean differences in the two distributions as described in the test statistic.

<iframe
  src="assets/empirical_diff_sugar.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

The **observed statistic** of **2.85** is indicated by the red vertical line on the graph. Since the **p_value** that we found **(0.0)** is < 0.05 which is the significance level that we set, we **reject the null hypothesis**. The missingness of `'rating'` does depend on the `'protein'` which suggests a MAR relationship between the two variables.

> Minutes and Rating

**Null Hypothesis:** The missingness of ratings does not depend on the length of cooking time (in minutes). 

**Alternate Hypothesis:** The missingness of ratings does depend on the length of cooking time (in minutes)

**Test Statistic:** The absolute difference of mean in cooking time of the recipe in minutes of the distribution of the group without missing ratings and the distribution of the group with missing ratings.

**Significance Level:** 0.05

<iframe
  src="assets/empirical_diff_prescale.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

Due to the outliers in cooking time, it is difficult to identify the shapes of the two distributions, so we update the scale to take a closer look.

<iframe
  src="assets/distr_rating_minutes.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

We ran another permutation test by shuffling the missingness of rating for 1000 times to collect 1000 simulating mean differences in the two distributions as described in the test statistic.

<iframe
  src="assets/empirical_diff_minutes.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

The **observed statistic** of **51.4524** is indicated by the red vertical line on the graph. Since the **p-value** that we found **(0.123)** is > 0.05 which is the significance level that we set, we **fail to reject the null hypothesis**. The missingness of rating does not depend on the cooking time in minutes of the recipe.


| Quarter     |   Count |
|:------------|--------:|
| Fall 2020   |       3 |
| Winter 2021 |       2 |
| Spring 2021 |       6 |
| Summer 2021 |       4 |
| Fall 2021   |      55 |

---

## Hypothesis Testing

In order to dive deeper into the relationship between protein content and recipes, we will perform a permuttion test with the following information:

- **Null Hypothesis (H₀):** There is no difference in the average user rating (as measured by the “average_rating” column) between high-protein and low-protein recipes
- **Alternative Hypothesis (H₁):** High-protein recipes receive significantly lower average ratings than low-protein recipes

**Test Statistic**

We want to test whether high-protein recipes get lower ratings, so we do a one-tailed, permutation test to test if the mean rating in the high-protein group is lower than the mean rating in the low-protein group. To do this we use th e difference in the means of **average_rating** between the two groups as our test statistic. Our reasoning for doing a difference of means is because we want to be able to highlight any directional findings.

**Method**
- **Observed Difference:** Mean (high-protein ratings) – Mean (low-protein ratings) 
- **Permutation Procedure:** we will randomly shuffle the protein-group labels many times (1,000 permutations) and recalculate the difference each time to build an empirical distribution
- **P-value:** we will calculate the p-value as the proportion of permuted differences that are less than or equal to the observed difference (if the observed difference is negative). 

Our goal is to see if protein and recipes come from the same population, thus our reasoning for performing a permutation test. 

After performing the permutation test, we reached an observed difference of -0.02 and accepted the null hypothesis.

## Framing a Prediction Problem