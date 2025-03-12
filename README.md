# Enhancing Recipe Quality Prediction through Nutritional Feature Engineering

by Edwin Ruiz & Alexa Fernandez Tobias


---

## Introduction

As people have become more health concsious over the years, the protein content in the foods we consume has become a popular topic of discussion. Those that are extremely interested in the gym and fitness seem to obssess over their protein goals and often eat more nutritionally dense, but overall less satisfactory meals. On the other hand, the average person not overly-critical of the foods they consume tend to gravitate to lower-protein meals that (objectively) taste better. We wonder if the ratings of high-protein meals recieve lower average ratings because of how they may be associated with nourishing and nutritious, though less appetizing, meals.

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

2. Converte the submitted and date columns to datetime datatype

3. Replace ratings of 0 with NaN. 
    Ratings of 0 indicate that someone did not rate the food. In order to avoid a drop in average ratings, we substitute 0 with NaN which avoids all 0's in future calculations.

4. Parse the nutrition column
    Orignally, this column is a string structured like a list. We convert each nutritional value in the String to an Integer in order to be able to perform further analysis. 

5. Concat new columns from the nutrition column to our merged dataframe.

6. Grouping by recipe id, we computed the average rating per recipe using our merged data. We then merged the resulting dataframe with the original merged_df to add 'avg_ratings' as a new column.

Here are the first 5 (unique) rows of our cleaned dataset:

| name                                 |     id |   minutes | submitted           |   rating |   average rating |   calories (#) |   sugar (PDV) | is_dessert   |   prop_sugar |
|:-------------------------------------|-------:|----------:|:--------------------|---------:|-----------------:|---------------:|--------------:|:-------------|-------------:|
| 1 brownies in the world    best ever | 333281 |        40 | 2008-10-27 00:00:00 |        4 |                4 |          138.4 |            50 | True         |    0.361272  |
| 1 in canada chocolate chip cookies   | 453467 |        45 | 2011-04-11 00:00:00 |        5 |                5 |          595.1 |           211 | False        |    0.354562  |
| 412 broccoli casserole               | 306168 |        40 | 2008-05-30 00:00:00 |        5 |                5 |          194.8 |             6 | False        |    0.0308008 |
| millionaire pound cake               | 286009 |       120 | 2008-02-12 00:00:00 |        5 |                5 |          878.3 |           326 | True         |    0.371172  |
| 2000 meatloaf                        | 475785 |        90 | 2012-03-06 00:00:00 |        5 |                5 |          267   |            12 | False        |    0.0449438 |


**Distribution of Recipe Protein Content** 
We implemented this plot to get a broader idea of the protein distribution. 
<iframe
  src="assets/univariate.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>



---

## Assessment of Missingness

Here's what a Markdown table looks like. Note that the code for this table was generated _automatically_ from a DataFrame, using

```py
print(counts[['Quarter', 'Count']].head().to_markdown(index=False))
```

| Quarter     |   Count |
|:------------|--------:|
| Fall 2020   |       3 |
| Winter 2021 |       2 |
| Spring 2021 |       6 |
| Summer 2021 |       4 |
| Fall 2021   |      55 |

---

## Hypothesis Testing


---