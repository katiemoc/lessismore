# **Less is More - Recipe Ingredient Analysis**

Author: Katie Moc

## Introduction

The saying "less is more" recognizes the importance of simplicity and clarity from an artistic perspective. When it comes to the art of cooking, one might argue that the simplest and most natural recipes are better than complex creations-- not just in taste, but also in health, preparation time, and even comfort.

The following data science project, conducted at UC San Diego, explores recipes and reviews on <a href='https://food.com'>food.com</a>, from 2008 to 2018, to answer the following question: **What is the relationship between the number of ingredients and average rating of a recipe?** In particular, as the title of this project suggests, I want to find out if *less is more*-- that is, if recipes with less ingredients have higher overall ratings than those with more.

From <a href='https://food.com'>food.com</a>, I will use two datasets-- one containing recipes, the other with their ratings-- which we will merge for our analysis. The following tables define the columns of each dataset.

**Dataset #1: `recipes` - 83782 rows (unique recipes), 12 columns**
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

**Dataset #2: `interactions` - 731927 rows (unique reviews), 5 columns**
| Column | Description |
| ------ | ----------- |
| `'user_id'` | User ID who submitted this reviewing |
| `'recipe_id'` | Recipe ID that user is reviewing |
| `'date'` | Date review was submitted |
| `'rating'` | Rating given |
| `'review'` | Review text |

Though counterintuitive, this question 

## Data Cleaning and Exploratory Data Analysis

### Data Cleaning

To allow for easy analysis, I first will execute the following cleaning steps, in this order:

1. Left merge the **`recipes`** dataset with **`interactions`**, by `id` and `recipe_id`.
    - Each recipe and their review(s) are aligned in the same row(s). 
    - All recipes will be kept, regardless of if they have any reviews. Not all reviews are kept.
    - Drop `id` to remove repetition.

2. In the `rating` column, reviews that chose not to leave a rating default to 0. Replace these 0's with `NaN`.
    - Prevents miscalculations in this column. 
        - For instance, if we found the mean of the `rating` column with these 0's, the mean would be biased low.

3. Find the average (mean) rating of each recipe and create `avg_rating` column. 
    - Recipes with multiple ratings can be generalized by their average.
    - Recipes with no ratings have an `avg_rating` of `NaN`.

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

5. Convert the `submitted` and `date` columns to `datetime` objects.
    - This conversion allows us to look at the data over time, from 2008 to 2018.
    - Rename to `recipe_date` and `review_date`, respectively, for clarity.

6. Convert the `tags`, `nutrition`, `steps`, and `ingredients` columns to lists.
    - This conversion allows us to iterate through each value.
    - `tags`, `steps`, and `ingredients` contain lists of strings.
    - `nutrition` contains lists of numbers (floats).

7. Consider a recipe to be "simple" if it has at most 5 ingredients. Create a new column `simple`.
    - Looks at `n_ingredients` and returns a boolean value.
        - `True` if the recipe has more than 5 ingredients.
        - `False` if the recipe has 5 or less ingredients.
    - Helpful for future hypothesis testing.

The column data types and first five *unique* rows of the newly cleaned DataFrame, **`merged`**, are shown below:

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
| `'simple'` | bool |

Note that not all columns in the table are shown for clarity-- I chose the most relevant ones. Scroll right to view more.

| recipe_id | name | minutes | recipe_date | n_steps | tags | ingredients | n_ingredients | review_date | rating | avg_rating | simple |
|----------:|:-----|--------:|:------------|--------:|:-----|:------------|--------------:|:------------|-------:|-----------:|:-------|
| 333281 | 1 brownies in the world best ever | 40 | 2008-10-27 00:00:00 | 10 | ['60-minutes-or-less', 'time-to-make', 'course', 'main-ingredient', ...] | ['bittersweet chocolate', 'unsalted butter', 'eggs', 'granulated sugar', ...] | 9 | 2008-11-19 00:00:00 | 4 | 4 | True |
| 453467 | 1 in canada chocolate chip cookies | 45 | 2011-04-11 00:00:00 | 12 | ['60-minutes-or-less', 'time-to-make', 'cuisine', 'preparation', ...] | ['white sugar', 'brown sugar', 'salt', 'margarine', 'eggs', 'vanilla', ...] | 11 | 2012-01-26 00:00:00 | 5 | 5 | True |
| 306168 | 412 broccoli casserole | 40 | 2008-05-30 00:00:00 | 6 | ['60-minutes-or-less', 'time-to-make', 'course', 'main-ingredient', 'preparation', ...] | ['frozen broccoli cuts', 'cream of chicken soup', 'sharp cheddar cheese', ...] | 9 | 2008-12-31 00:00:00 | 5 | 5 | True |
| 286009 | millionaire pound cake | 120 | 2008-02-12 00:00:00 | 7 | ['time-to-make', 'course', 'cuisine', 'preparation', 'occasion', 'north-american', ...] | ['butter', 'sugar', 'eggs', 'all-purpose flour', 'whole milk', ...] | 7 | 2008-04-09 00:00:00 | 5 | 5 | True |
| 475785 | 2000 meatloaf | 90 | 2012-03-06 00:00:00 | 17 | ['time-to-make', 'course', 'main-ingredient', 'preparation', 'main-dish', 'potatoes', ...] | ['meatloaf mixture', 'unsmoked bacon', 'goat cheese', 'unsalted butter', 'eggs', ...] | 13 | 2012-03-07 00:00:00 | 5 | 5 | True |



### Univariate Analysis
<iframe
  src="assets/n_ingredients_hist.html"
  width="1000"
  height="500"
  frameborder="0"
></iframe>

### Bivariate Analysis
<iframe
  src="assets/rating_cond_hist.html"
  width="1000"
  height="500"
  frameborder="0"
></iframe>

### Interesting Aggregates

## Assessment of Missingness

## Hypothesis Testing

## Framing a Prediction Problem

## Baseline Model

## Final Model

## Fairness Analysis