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
    - Looks at `'n_ingredients'` and returns a boolean value.
        - `True` if the recipe has more than 5 ingredients.
        - `False` if the recipe has 5 or less ingredients.
    - Helpful for future hypothesis testing.

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
 
**Cleaned Dataset: `merged`** - 234429 rows, 18 columns
> Note: not all columns in the table are shown for clarity-- I chose the most relevant ones. Scroll right to view more.

| recipe_id | name | minutes | recipe_date | n_steps | tags | ingredients | n_ingredients | review_date | rating | avg_rating | simple |
|----------:|:-----|--------:|:------------|--------:|:-----|:------------|--------------:|:------------|-------:|-----------:|:-------|
| 333281 | 1 brownies in the world best ever | 40 | 2008-10-27 00:00:00 | 10 | ['60-minutes-or-less', 'time-to-make', 'course', ...] | ['bittersweet chocolate', 'unsalted butter', 'eggs', ...] | 9 | 2008-11-19 00:00:00 | 4 | 4 | True |
| 453467 | 1 in canada chocolate chip cookies | 45 | 2011-04-11 00:00:00 | 12 | ['60-minutes-or-less', 'time-to-make', 'cuisine', ...] | ['white sugar', 'brown sugar', 'salt', ...] | 11 | 2012-01-26 00:00:00 | 5 | 5 | True |
| 306168 | 412 broccoli casserole | 40 | 2008-05-30 00:00:00 | 6 | ['60-minutes-or-less', 'time-to-make', 'course', ...] | ['frozen broccoli cuts', 'cream of chicken soup', ...] | 9 | 2008-12-31 00:00:00 | 5 | 5 | True |
| 286009 | millionaire pound cake | 120 | 2008-02-12 00:00:00 | 7 | ['time-to-make', 'course', 'cuisine', 'preparation', ...] | ['butter', 'sugar', 'eggs', ...] | 7 | 2008-04-09 00:00:00 | 5 | 5 | True |
| 475785 | 2000 meatloaf | 90 | 2012-03-06 00:00:00 | 17 | ['time-to-make', 'course', 'main-ingredient', ...] | ['meatloaf mixture', 'unsmoked bacon', 'goat cheese', ...] | 13 | 2012-03-07 00:00:00 | 5 | 5 | True |

### Univariate Analysis

The distribution of the `n_ingredients` column gives us insight of how many recipes are "simple." The graph appears normal with a right skew-- showing a few recipes with a large amount of ingredients, with the maximum being 35! This figure also shows that more recipes posted on <a href='https://food.com'>food.com</a> have more than 5 ingredients. The number of recipes and reviews increase as the number of ingredients increase from 1 to 8, while they start decreasing from 9 onwards.

<iframe
  src="assets/n_ingredients_hist.html"
  width="1000"
  height="500"
  frameborder="0"
></iframe>

To contrast, the distribution of the `'avg_rating'` column shows us how recipes are typically rated. The figure has a left skew, with many more recipes having an average rating between 4 and 5. The number of recipes and reviews increase as rating increases.

<iframe
  src="assets/avg_rating_hist.html"
  width="1000"
  height="500"
  frameborder="0"
></iframe>

### Bivariate Analysis

The distribution of the `'rating'` column, conditioned on `'is_simple'`, compares the ratings of "simple" recipes (up to 5 ingredients) and "complex" recipes (more than 5 ingredients). While the overall distributions are quite similar, "simple" recipes show a slightly higher proportion of 4 and 5-star ratings compared to their "complex" counterparts.

<iframe
  src="assets/rating_cond_hist.html"
  width="1000"
  height="500"
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

It is intuitive to believe that as the the number of ingredients increase, so would the number of steps. The figure below follows this belief, as there is a slight increase in the min, mean, and median lines as the number of ingredients increase. These three aggregates share a similar pattern near the lower end of the graph, showing that majority of recipes, regardless of their ingredient count, have around 20 or less steps. However, the max line fluctuates between 50 and 100 throughout the entire graph, highlighting that any number of ingredients may still require many steps. 

The decline of all four aggregate lines at the right end of the graph can be explained by the fact that there are some ingredient counts with only one recipe match. This would cause all four aggregates to be the same value.

<iframe
  src="assets/agg_line.html"
  width="1000"
  height="500"
  frameborder="0"
></iframe>



## Assessment of Missingness




## Hypothesis Testing




## Framing a Prediction Problem




## Baseline Model




## Final Model




## Fairness Analysis



