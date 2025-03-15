# Less is More - Recipe Ingredient Analysis

**Author:** Katie Moc

## Introduction

The saying "less is more" recognizes the importance of simplicity and clarity from an artistic perspective. When it comes to the art of cooking, one might argue that the most simple and natural recipes are better than complex creations-- not just in taste, but also in health, preparation time, and even comfort.

The following data science project, conducted at UC San Diego, explores recipes and reviews on <a href='https://food.com'>food.com</a>, from 2008 to 2018, to answer the following question: **What is the relationship between the number of ingredients and average rating of a recipe?** In particular, as the title of this project suggests, it aims to find out if *less is more*-- that is, if recipes with less ingredients have higher overall ratings.

From <a href='https://food.com'>food.com</a>, two datasets-- one containing recipes, the other with their ratings-- are merged for this analysis. The following tables define the columns of each dataset.

**Dataset #1: `recipes`** - 83782 rows (unique recipes), 12 columns

| Column | Description |
| -------| ----------- |
| `'name'` | Recipe name |
| `'id'` | Recipe ID |
| `'minutes'` | Minutes to prepare recipe |
| `'contributer_id'` | User ID who submitted this recipe |
| `'submitted'` | Date recipe was submitted |
| `'tags'` | food.com tags for recipe |
| `'nutrition'` | Nutrition information in the form [calories (#), total fat (PDV), sugar (PDV), sodium (PDV), protein (PDV), saturated fat (PDV), carbohydrates (PDV)]; PDV stands for “percentage of daily value” |
| `'n_steps'` | Number of steps in recipe |
| `'steps'` | Steps to prepare recipe |
| `'description'` | Text for recipe steps, in order |
| `'ingredients'` | Ingredient list |
| `'n_ingredients'` | Number of ingredients in recipe |

**Dataset #2: `interactions`** - 731927 rows (unique reviews), 5 columns

| Column | Description |
| ------ | ----------- |
| `'user_id'` | User ID who submitted this reviewing |
| `'recipe_id'` | Recipe ID that user is reviewing |
| `'date'` | Date review was submitted |
| `'rating'` | Rating given |
| `'review'` | Review text |

This insight could be valuable to the <a href='https://food.com'>food.com</a> community, helping them determine whether straightforward, minimal-ingredient recipes or more elaborate, intricate ones have gained popularity in recent years. By identifying these trends, they can strategically post and promote recipes that align with current audience preferences, ultimately increasing engagement on their platform and driving revenue growth.

Beyond its practical applications, this question also challenges the conventional belief that *more* equates to *better*. In the modern era of social media, with aesthetics heavily influencing our views and opinions, the food we cook (or simply, the ones we feel are "post-worthy") have been shaped by visual appeal rather than taste or tradition. A greater appreciation for simplicity in cooking could spark a change in culture-- one that values the comfort, nostalgia, and authenticity of a beloved meal over its complexity. Understanding these evolving dynamics can offer deeper insights into how digital trends reshape our relationship with food.



## Data Cleaning and Exploratory Data Analysis

### Data Cleaning

To allow for easy analysis, the following cleaning steps were executed, in this order:

1. Left merge the **`recipes`** dataset with **`interactions`**, by `'id'` and `'recipe_id'`.
    - Each recipe and their review(s) are aligned in the same row(s). 
    - All recipes will be kept, regardless of if they have any reviews. Not all reviews are kept.
    - Drop `'id'` to remove repetition.

2. In the `'rating'` column, reviews that chose not to leave a rating default to 0. Replace these 0's with `NaN`.
    - Prevents miscalculations in this column. 
        - For instance, if we found the mean of the `'rating'` column with these 0's, the mean would be biased low.

3. Find the average (mean) rating of each recipe and create `'avg_rating'` column. 
    - Recipes with multiple ratings can be generalized by their average.
    - Recipes with no ratings have an `'avg_rating'` of `NaN`.

4. Check data types of each column to see what conversions are needed.

    | Column | Data Type |
    |--------|-----------|
    | `'name'` | object |
    | `'minutes'` | int64 |
    | `'contributer_id'` | int64 |
    | `'submitted'` | object |
    | `'tags'` | object |
    | `'nutrition'` | object |
    | `'n_steps'` | int64 |
    | `'steps'` | object |
    | `'description'` | object |
    | `'ingredients'` | object |
    | `'n_ingredients'` | int64 |
    | `'user_id'` | float64 |
    | `'recipe_id'` | float64 |
    | `'date'` | object |
    | `'rating'` | float64 |
    | `'review'` | object |
    | `'avg_rating'` | float64 |

5. Convert the `'submitted'` and `'date'` columns to `datetime` objects.
    - This conversion allows us to look at the data over time, from 2008 to 2018.
    - Rename to `'recipe_date'` and `'review_date'`, respectively, for clarity.

6. Convert the `'tags'`, `'nutrition'`, `'steps'`, and `'ingredients'` columns to lists.
    - This conversion allows us to iterate through each value.
    - `'tags'`, `'steps'`, and `'ingredients'` contain lists of strings.
    - `'nutrition'` contains lists of numbers.

7. Consider a recipe to be "simple" if it has at most 5 ingredients. Create a new column `'is_simple'`.
    - Helpful for future hypothesis testing.
    - Looks at `'n_ingredients'` and returns a boolean value.
        - `True` if the recipe has more than 5 ingredients.
        - `False` if the recipe has 5 or less ingredients.

The column data types and first 5 *unique* rows of the newly cleaned DataFrame, **`merged`**, are shown below. 

| Column | Data Type |
|--------|-----------|
| `'name'` | object |
| `'minutes'` | int64 |
| `'contributer_id'` | int64 |
| `'recipe_date'` | datetime64[ns] |
| `'tags'` | object |
| `'nutrition'` | object |
| `'n_steps'` | int64 |
| `'steps'` | object |
| `'description'` | object |
| `'ingredients'` | object |
| `'n_ingredients'` | int64 |
| `'user_id'` | float64 |
| `'recipe_id'` | float64 |
| `'review_date'` | datetime64[ns] |
| `'rating'` | float64 |
| `'review'` | object |
| `'avg_rating'` | float64 |
| `'is_simple'` | bool |
 
**Cleaned Dataset: `merged`** - 234429 rows, 18 columns (*Note: Only the most relevant columns are shown, for clarity. Scroll right to view more.*)

| recipe_id | name | minutes | recipe_date | n_steps | tags | ingredients | n_ingredients | review_date | rating | avg_rating | simple |
|----------:|:-----|--------:|:------------|--------:|:-----|:------------|--------------:|:------------|-------:|-----------:|:-------|
| 333281 | 1 brownies in the world best ever | 40 | 2008-10-27 00:00:00 | 10 | ['60-minutes-or-less', 'time-to-make', 'course', ...] | ['bittersweet chocolate', 'unsalted butter', 'eggs', ...] | 9 | 2008-11-19 00:00:00 | 4 | 4 | True |
| 453467 | 1 in canada chocolate chip cookies | 45 | 2011-04-11 00:00:00 | 12 | ['60-minutes-or-less', 'time-to-make', 'cuisine', ...] | ['white sugar', 'brown sugar', 'salt', ...] | 11 | 2012-01-26 00:00:00 | 5 | 5 | True |
| 306168 | 412 broccoli casserole | 40 | 2008-05-30 00:00:00 | 6 | ['60-minutes-or-less', 'time-to-make', 'course', ...] | ['frozen broccoli cuts', 'cream of chicken soup', ...] | 9 | 2008-12-31 00:00:00 | 5 | 5 | True |
| 286009 | millionaire pound cake | 120 | 2008-02-12 00:00:00 | 7 | ['time-to-make', 'course', 'cuisine', 'preparation', ...] | ['butter', 'sugar', 'eggs', ...] | 7 | 2008-04-09 00:00:00 | 5 | 5 | True |
| 475785 | 2000 meatloaf | 90 | 2012-03-06 00:00:00 | 17 | ['time-to-make', 'course', 'main-ingredient', ...] | ['meatloaf mixture', 'unsmoked bacon', 'goat cheese', ...] | 13 | 2012-03-07 00:00:00 | 5 | 5 | True |

### Univariate Analysis

The distribution of the `'n_ingredients'` column gives us insight of how many recipes are "simple." The graph appears normal with a right skew-- showing a few recipes with a large amount of ingredients, with the maximum being 35! This figure also shows that more recipes posted on <a href='https://food.com'>food.com</a> have more than 5 ingredients. The number of recipes and reviews increase as the number of ingredients increase from 1 to 8, while they start decreasing from 9 onwards.

<iframe
  src="assets/n_ingredients_hist.html"
  width="600"
  height="400"
  frameborder="0"
></iframe>

To contrast, the distribution of the `'avg_rating'` column shows us how recipes are typically rated. The figure has a left skew, with many more recipes having an average rating between 4 and 5. The number of recipes and reviews increase as rating increases.

<iframe
  src="assets/avg_rating_hist.html"
  width="600"
  height="400"
  frameborder="0"
></iframe>

### Bivariate Analysis

The distribution of the `'rating'` column, conditioned on `'is_simple'`, compares the ratings of "simple" recipes (up to 5 ingredients) and "complex" recipes (more than 5 ingredients). While the overall distributions are quite similar, "simple" recipes show a slightly higher proportion of 4 and 5-star ratings compared to their "complex" counterparts.

<iframe
  src="assets/rating_cond_hist.html"
  width="600"
  height="400"
  frameborder="0"
></iframe>

### Interesting Aggregates

The aggregates between the `'n_ingredients'` and `'n_steps'` columns highlight an interesting relationship. The following smaller DataFrame groups the recipes by their number of ingredients and finds the mean, median, minimum, and maximum number of steps of each group. The first 10 (out of 34) rows are shown.

|   n_ingredients |     mean |   median |   min |   max |
|----------------:|---------:|---------:|------:|------:|
|               1 |  6.75    |        7 |     2 |    20 |
|               2 |  6.4726  |        5 |     1 |    55 |
|               3 |  5.70518 |        5 |     1 |    69 |
|               4 |  6.64585 |        5 |     1 |    81 |
|               5 |  7.80237 |        6 |     1 |    80 |
|               6 |  7.79357 |        7 |     1 |    86 |
|               7 |  8.3636  |        8 |     1 |    55 |
|               8 |  9.09987 |        8 |     1 |    67 |
|               9 |  9.81518 |        9 |     1 |    88 |
|              10 | 10.6835  |       10 |     1 |    57 |

It is intuitive to believe that as the number of ingredients increase, so would the number of steps. The figure below follows this belief, as there is a slight increase in the min, mean, and median lines as the number of ingredients increase. These three aggregates share a similar pattern near the lower end of the graph, showing that majority of recipes, regardless of their ingredient count, have around 20 or less steps. However, the max line fluctuates between 50 and 100 throughout the entire graph, highlighting that any number of ingredients may still require many steps. 

The decline and overlap of all four aggregate lines at the right end of the graph can be explained by the fact that there are some ingredient counts with only one recipe match. This causes all four aggregates to be the same value.

<iframe
  src="assets/agg_line.html"
  width="900"
  height="500"
  frameborder="0"
></iframe>



## Assessment of Missingness

Four columns in the **`merged`** DataFrame have missing values: `'description'`, `'rating'`, `'review'`, and `'avg_rating'`. The missingness in the `'avg_rating'` column is missing by design since it strictly uses information from the `'rating'` column. Therefore, this section explores the missingness of the `'rating'` column.

| Column | Number of Missing Values |
|--------|--------------------------|
| `'description'` | 114 |
| `'rating'` | 15036 |
| `'review'` | 58 |
| `'avg_rating'` | 2777 |

### NMAR Analysis

The missingness in the `'review'` column is arguably not missing at random (NMAR) for one main reason: people simply do not feel like leaving reviews on recipes they do not care for. Reviewers are more likely to express their opinions and spend their time writing a review only if they feel strongly about how good or bad a recipe is. People also need to be interested enough to make the recipe in the first place. Hence, the `'review'` column is missing when someone chooses not to explain their rating, or if a recipe has no ratings at all. This information could be utilized to identify unpopular recipes and what should be done to either improve or promote these recipes.

### Missingness Dependency

The missingness of the `'rating'` column will be tested on the `'minutes'` and `'n_ingredients'` columns. Since the distribution of the missing values will be compared to the distribution of non-missing values, this requires permutation testing. The following steps were executed:

1. Create a new column `'rating_missing'` to store boolean values.
    - True if `'rating'` is missing (`NaN`).
    - False if `'rating'` is not missing.

2. Explore the distributions of `'minutes'` and `'n_ingredients'`, conditioned on `'rating_missing'`.
    - Helps us determine what test statistic to use.

    For `'minutes'`, the distributions are very similar and both show a decreasing trend. As the number of minutes increase, the proportion of recipes decrease, regardless of rating missingness. Additionally, a larger proportion of recipes with missing ratings have smaller completion times, while recipes with ratings have larger completion times.

    <iframe
    src="assets/min_missing_fig.html"
    width="600"
    height="400"
    frameborder="0"
    ></iframe>

    For `'n_ingredients'`, the distributions are again very similar, but follow a bell curve with right skew. As the number of ingredients increase, the proportion of recipes also increase until peaking at 9 and decreasing after that. Though slight, a larger proportion of recipes with missing ratings have less ingredients, while recipes with ratings have more ingredients.

    <iframe
    src="assets/ingr_missing_fig.html"
    width="600"
    height="400"
    frameborder="0"
    ></iframe>

    Since both of the figures above show similar distributions, the test statistic of the permutation tests will be absolute difference in group means.

3. Perform permutation tests, using **absolute difference in group means** and significance level **0.05**.

    **Minutes and Rating:**
    - **Null Hypothesis:** The missingness of the `'rating'` of a recipe depends on the time it takes to complete.
    - **Alternative Hypothesis:** The missingness of the `'rating'` of a recipe does not depend on the time it takes to complete.

    After shuffling the `'rating_missing'` column and calculating the absolute difference in means 1000 times, the distribution of differences, compared to the observed value, is shown below:

    <iframe
    src="assets/min_test_fig.html"
    width="800"
    height="400"
    frameborder="0"
    ></iframe>

    **Conclusion**: The observed statistic is **51.452**. The p-value is **0.123**, which is **larger** than 0.05. Thus, we **fail to reject** the null hypothesis and do **not** have significant evidence that the missingness of the `'ratings'` column depends on the completion time of a recipe.

    **Number of Ingredients and Rating:**
    - **Null Hypothesis:** The missingness of the `'rating'` of a recipe depends on the number of ingredients it takes to make.
    - **Alternative Hypothesis:** The missingness of the `'rating'` of a recipe does not depend on the number of ingredients it takes to make.

    After shuffling the `'rating_missing'` column and calculating the absolute difference in means 1000 times, the distribution of differences, compared to the observed value, is shown below:

    <iframe
    src="assets/ingr_test_fig.html"
    width="800"
    height="400"
    frameborder="0"
    ></iframe>

    **Conclusion**: The observed statistic is **1.339**. The p-value is **0.0**, which is **smaller** than 0.05. Thus, we **reject** the null hypothesis and  have significant evidence that the missingness of the `'ratings'` column does depend on the number of ingredients in a recipe.



## Hypothesis Testing

**What is the relationship between the number of ingredients and average rating of a recipe?** This section performs a permutation test using the `'avg_rating'` and `'is_simple'` columns under the following hypotheses:
- **Null Hypothesis**: The average rating of recipes is the same for those with more than five ingredients and those with at most five ingredients.
- **Alternative Hypothesis**: Recipes with more than five ingredients have a lower average rating.

Since our alternative hypothesis indicates a direction, our test statistic will be the **signed difference in group means.** Our significance level is **0.05.**

After shuffling the `'is_simple'` column and calculating the mean difference 1000 times, the distribution of differences, compared to the observed value, is shown below:

<iframe
src="assets/simple_test_fig.html"
width="800"
height="400"
frameborder="0"
></iframe>

**Conclusion**: The observed statistic is **-0.037**. The p-value is **0.0**, which is **smaller** than 0.05. Thus, we **reject** the null hypothesis and have significant evidence that recipes with more than five ingredients have a lower average rating.



## Framing a Prediction Problem

Let us now create a model that **predicts the average rating** of a recipe. We will treat this as a **regression** problem, since the average rating is a quantitative continuous value between 1.0 and 5.0. Therefore, the response variable is the `'avg_rating'` column-- it is a fair representation of each recipe over all of its reviews. In the previous section, we found that there is a significant difference between the average rating of simple and complex recipes, so it will be interesting to explore this further.

We know from our exploratory data analysis that some relationships between columns and `'avg_rating'` do not show a linear trend. Therefore, for this model, we will use the **`RandomForestRegressor`**, along with the **mean absolute error** as the metric. Some columns have a large range of values, and the mean absolute error is more robust to outliers, so it is more ideal than mean squared error in this case.

Since we are predicting the average rating, the information known at the time of prediction are all columns from the original `recipe` dataset. This is because a person must read and create the recipe in order to properly rate it.



## Baseline Model

Earlier, we found that the missingness of the `'rating'` column does depend on the `'n_steps'` and `'n_ingredients'` columns. The baseline model will try to predict a recipe's `'avg_rating'` with the same two columns to discover if this missingness dependency might also help predict the average rating. Both of these columns contain **quantitative** values, so no encodings were applied. 

After splitting the data into training and testing sets, training the model with the random forest regressor, and calculating its predictions, the baseline model achieved a mean absolute error of **0.3356** on the training set, and **0.3386** on the test set. Since the two mean absolute errors are very, this baseline model is arguably "good" because it can generalize the data well, rather than over or underfitting it.


## Final Model

The goal of the final model is to **further minimize the mean absolute error** while also ensuring this new model can **generalize** the data well. We will add two more features: `'minutes'` and `'recipe_date'`. While we will transform these new features, the two original features, `'n_steps'` and `'n_ingredients'`, remain the same.

The relationship between the `'minutes'` and `'avg_rating'` columns shows that majority of high rated recipes have shorter completion times. This could improve our model by utilizing the difference in ratings for various completion times. Arguably, recipes that take longer to prepare suggest a larger probability of making a mistake. To this column, we apply a **`QuantileTransformer`** because its range of values is vast, from 0 to over 1 million minutes. This transformer maps the data into a normal distribution, so that the model is more robust to outliers. The transformed `'minutes'` column contains **quantitative** values.

The relationship between the `'recipe_date'` and `'avg_rating'` columns reveals that the average rating of recipes fluctuates from 2008 to 2014, but strictly declines from 2014 to 2018. This could improve our model by factoring in how recent the recipe was posted. To this column, we apply a **`FunctionTransformer`** so we can track the average recipe ratings over time. Through hyperparameter testing, the model will decide between two functions:

- `extract_year()`: simply extracts the year that the inputted recipe was posted
- `days_since()`: calculates the number of days between the earliest posted recipe, which is January 1st, 2008, and the inputted recipe.

The transformed `'recipe_date'` column contains **quantitative** values regardless of the chosen function.

Furthermore, hyperparameter testing will also help choose the `n_estimators` and `max_depth` of our `RandomForestRegressor`. This will ensure that the model, which utilizes decision trees, does not overfit the training data nor fail to generalize the data. Using `GridSearchCV`, the combination of hyperparameters that best minimizes the mean absolute error is the **`days_since` function, `max_depth = 20`, and `n_estimators = 75`**.

Under the same training and test sets as the baseline model, the new model achieved a mean absolute error of **0.2177** for the training set, and **0.2581** for the test set. The two mean absolute errors are again, relatively similar, with a difference of only about 0.04. The new model seems to generalize the data well, and has a lower mean absolute error than the baseline model! 



## Fairness Analysis

Our final model has improved, but is it fair? We will test the fairness of our final model on 'simple' and 'complex' recipes-- that is, recipes with 5 or less ingredients, compared to recipes with more than 5.

**Group X:** Recipes with 5 or less ingredients, or **simple** recipes

**Group Y:** Recipes with more than 5 ingredients, or **complex** recipes

**Evaluation Metric:** Root mean squared error, or **RMSE**
- The RMSE is a better choice than R-squared in this case because we want to test the accuracy of our predictions, rather than determine the explainability of a predictor variable.

We will utilize the `is_simple` column for our permutation test with the following hypotheses:
- **Null Hypothesis:** Our model is fair. Its RMSE for simple recipes and complex recipes are roughly the same, and any differences are due to random chance.
- **Alternative Hypothesis:** Our model is unfair. Its RMSE for simple recipes is significantly different than its RMSE for complex recipes.
- **Test Statistic:** Absolute difference in RMSE (\|simple - complex\|)
- **Significance Level:** 0.05

After shuffling the `'is_simple'` column and calculating the absolute difference in RMSE 1000 times, the distribution of differences, compared to the observed value, is shown below:

<iframe
src="assets/rmse_fig.html"
width="800"
height="400"
frameborder="0"
></iframe>

**Conclusion:** The observed statistic is **0.069**. Since our p-value, 0.0, is **less than 0.5**, we **reject** the null hypothesis and have **significant** evidence to believe our model is **not fair**. The model's RMSE for simple recipes is significantly different than its RMSE for complex recipes.
