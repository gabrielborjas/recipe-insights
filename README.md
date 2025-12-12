# Recipe Insights: Healthiness, Ratings, and Recipe Complexity

Authors: Gabriel Borjas-Perez & Jared Parra

## Introduction

If you’ve ever searched for a dinner idea online, you know how overwhelming it can be! Thousands of recipes, each claiming to be “easy,” “healthy,” or “five-star.” What’s much less obvious is how people actually respond to those recipes once they cook them. Do home cooks reward lighter, healthier dishes with high ratings, or do the richest, most indulgent recipes still win?

In this project, we investigate how the **healthiness** and **complexity** of recipes relate to how highly they are rated on Food.com. Our central question is:

> **How are recipe ratings related to both their nutritional profile (healthy vs. unhealthy)?**

To answer this, we use two raw datasets from Food.com: one containing recipe metadata and nutrition information, and another containing user interactions (ratings and reviews). Below, we describe each dataset and the columns that are most relevant to our analysis.

### The `recipe` dataset

The first dataset, `recipe`, contains 83,782 rows, indicating 83,782 unique recipes, with the following columns:

| Column         | Description                                                                                     |
|----------------|-------------------------------------------------------------------------------------------------|
| `name`         | Name of the recipe                                                                              |
| `id`           | Unique recipe ID                                                                                |
| `minutes`      | Number of minutes required to prepare the recipe                                               |
| `contributor_id` | User ID of the person who originally submitted the recipe                                     |
| `submitted`    | Date the recipe was submitted to Food.com                                                      |
| `tags`         | Food.com tags associated with the recipe (e.g., cuisine, course, dietary labels)              |
| `nutrition`    | List of nutrition information in the form `[calories, total fat, sugar, sodium, protein, saturated fat, carbohydrates]` (all expressed as % Daily Value) |
| `n_steps`      | Number of steps in the recipe instructions (a key measure of recipe complexity in our project) |
| `steps`        | Text of the step-by-step instructions                                                          |
| `description`  | User-provided description or summary of the recipe                                             |
| `ingredients`  | Text listing the ingredients used in the recipe                                                |
| `n_ingredients`| Number of ingredients used in the recipe                                                       |

From this dataset, we derive numerical measures of **healthiness** (by unpacking the `nutrition` list into separate columns like calories, fat, sugar, protein, and sodium) and **complexity** (using `n_steps`).

### The `interactions` dataset

The second dataset, `interactions`, contains 731,927 rows, where each row represents a user’s interaction with a specific recipe. Its columns are:

| Column      | Description                                      |
|-------------|--------------------------------------------------|
| `user_id`   | ID of the user who rated or reviewed the recipe |
| `recipe_id` | ID of the recipe that the user interacted with  |
| `date`      | Date of the interaction                         |
| `rating`    | Rating given by the user (on a 1–5 scale)       |
| `review`    | Free-text review written by the user            |

To study these questions, we combined the two raw files into a single working dataset. We first renamed the recipe ID column in `RAW_recipes.csv` to `recipe_id` and merged it with `RAW_interactions.csv` on this key, so each recipe is linked to all of its reviews. Ratings of 0 were treated as missing (they don’t represent a real score), and we created an `avg` column that stores the average rating for each recipe.

The original `nutrition` column is stored as a stringified list, so we split it into separate numeric columns for key nutrients: `calories`, `total_fat`, `sugar`, `sodium`, `protein`, `saturated_fat`, and `carbs`. Along with these, we keep measures of recipe complexity such as `n_steps` (and later, `minutes` and `n_ingredients`).

The resulting merged dataset gives us, for every recipe, its nutritional profile, its complexity, and its average user rating—exactly what we need to investigate how healthiness and complexity relate to how people rate recipes on Food.com.

## Data Cleaning and Exploratory Data Analysis

1. Merge the two original datasets on `recipe_id`
   - This lets us study both the recipe info and the user ratings in one place.
     
     
2. Replace the ratings of 0 with missing values.
   - A rating of 0 doesn’t behave like a real score on the 1–5 scale, so we replaced 0s in `ratig` with `NaN`. This keeps those entries from dragging down averages in a misleading way.


3. Compute an average rating per recipe.
   - We grouped by `recipe_id` to calculate the mean of `rating` for each recipe and stored it in a new column, `avg`. This gives us a single and simple rating for each recipe.

  
4. Break the nutrition list column into separate numerical columns for each nutrient.
   - The original `nutrition` column is a string that stores a list of values. We cleaned the brackets and split it into seven numbers, then created the following columns:
     - `calories`
     - `total_fat`
     - `sugar`
     - `sodium`
     - `protein`
     - `saturated_fat`
     - `carbs`  
   - Turning this list into separate columns makes it much easier to analyze the different amount of specific nutrients are in each recipe.


Here is the head of our resulting cleaned DataFrame:

| recipe_id | nutrition                                                        | minutes | n_steps | ingredients                                                                                                                                                                    | rating | avg | calories | total_fat | sugar | sodium | protein | saturated_fat | carbs |
|----------:|:-----------------------------------------------------------------|--------:|--------:|:-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|-------:|----:|---------:|----------:|------:|-------:|--------:|--------------:|------:|
| 333281    | ['138.4', ' 10.0', ' 50.0', ' 3.0', ' 3.0', ' 19.0', ' 6.0']     | 40      | 10      | ['bittersweet chocolate', 'unsalted butter', 'eggs', 'granulated sugar', 'unsweetened cocoa powder', 'vanilla extract', 'brewed espresso', 'kosher salt', 'all-purpose flour'] | 4      | 4  | 138.4    | 10       | 50    | 3      | 3       | 19             | 6    |
| 453467    | ['595.1', ' 46.0', ' 211.0', ' 22.0', ' 13.0', ' 51.0', ' 26.0'] | 45      | 12      | ['white sugar', 'brown sugar', 'salt', 'margarine', 'eggs', 'vanilla', 'water', 'all-purpose flour', 'whole wheat flour', 'baking soda', 'chocolate chips']                    | 5      | 5  | 595.1    | 46       | 211   | 22     | 13      | 51             | 26   |
| 306168    | ['194.8', ' 20.0', ' 6.0', ' 32.0', ' 22.0', ' 36.0', ' 3.0']    | 40      | 6       | ['frozen broccoli cuts', 'cream of chicken soup', 'sharp cheddar cheese', 'garlic powder', 'ground black pepper', 'salt', 'milk', 'soy sauce', 'french-fried onions']          | 5      | 5  | 194.8    | 20       | 6     | 32     | 22      | 36             | 3    |
| 306168    | ['194.8', ' 20.0', ' 6.0', ' 32.0', ' 22.0', ' 36.0', ' 3.0']    | 40      | 6       | ['frozen broccoli cuts', 'cream of chicken soup', 'sharp cheddar cheese', 'garlic powder', 'ground black pepper', 'salt', 'milk', 'soy sauce', 'french-fried onions']          | 5      | 5  | 194.8    | 20       | 6     | 32     | 22      | 36             | 3    |
| 306168    | ['194.8', ' 20.0', ' 6.0', ' 32.0', ' 22.0', ' 36.0', ' 3.0']    | 40      | 6       | ['frozen broccoli cuts', 'cream of chicken soup', 'sharp cheddar cheese', 'garlic powder', 'ground black pepper', 'salt', 'milk', 'soy sauce', 'french-fried onions']          | 5      | 5  | 194.8    | 20       | 6     | 32     | 22      | 36             | 3    |


This histogram shows how calories are distributed across recipes. Most recipes fall in the 100–300 calorie range, and the bar heights drop off as calories increase, which tells us there are only a small number of extremely high-calorie recipes. However, we also see that there is not many recipes that are less than 100 calories.

<iframe
  src="assets/calories_hist.html"
  width="800"
  height="450"
  frameborder="0">
</iframe>
# Baseline Model

The model we chose to do was a linear regression model, since we wanted to predict the number of steps a recipe would take based on numerous factors. For the baseline model we looked at three different features to predict the number of steps, those being minutes, number of ingredients, and whether or not the recipe was healthy. Minutes and number of ingredients were both qualitative data, minutes was given in the dataset as a numerical feature so no changes had to be made. But for number of ingredients, we were given a list of the ingredients in the recipe and just transformed it into the length of the list to see how many ingredients there were in the recipe. We also used the healthy feature which is ordinal data, we used One Hot Encoding to be able to use the feature in our model, so it was just one column with a value of one indicating healthiness and 0 indicating non healthiness. Overall, this model wasn’t horrible, but it did not perform the best based on the RMSE. Our RMSE was 5.871, indicating that the projected number of steps for a recipe was on average almost 6 steps away from the real number of steps, so it did not have the best performance.

# Final Model

One of the new features we added was calories, a numerical value from the original dataset, we did not perform any transformations as the feature was already in numerical format. We added calories because we predict higher calorie recipes may contain more complexity in the recipe and thus have more steps. We also edited a previous feature ‘minutes’, we took the log value of each value to help stabilize the variance, making long recipes more comparable with shorter ones. The last feature we added was minutes per ingredient because higher “minutes per ingredient” suggests procedures like marinating, sautéing, baking, layering, etc., which typically involve multiple steps. We selected RandomForestRegressor for the final model because it can handle nonlinear relationships, which are common in recipe structure, and it works well with mixed feature types (numeric + encoded text). We performed hyperparameter tuning using GridSearchCV, evaluating models using a 3-fold cross-validation scheme. The grid included, n_estimators: number of trees in the forest, max_depth: how deep each tree can grow (controls overfitting), min_samples_leaf: minimum number of samples in a leaf (controls tree smoothness). These settings were chosen because they directly control the bias–variance tradeoff of the model. Overall the model performed better than the baseline model. While it is still may not be the most ideal model, it is an improvement. The RMSE of our final model was around 5.427, which is a significant improvement from the baseline model which had an RMSE of nearly 6. On top of that it takes more factors which influence the number of steps in a recipe, which is another reason for this being an improved model.

# Fairness Analysis

For the groups we chose Healthy (X, 1) vs Unhealthy (Y, 0). We chose these because healthiness could impact how predictable the number of steps a recipe takes, healthy recipes may be simpler and unhealthy recipes may have more complex steps for things like baked goods. Our evaluation metric was Root Mean Squared Error since this is a regression problem, RMSE measures how close the predictions are to the actual number of steps. Our Null hypothesis would be that the model is fair and the number of steps for healthy and unhealthy recipes is the same on average. Our alternate hypothesis is that model is unfair against healthy recipes, and performs higher for unhealthy recipes. Our test statistic was the RMSE for healthy recipes - RMSE for unhealthy recipes, with an observed value of -0.910, showing the model actually performs better for healthy recipes. Our significance value was 0.05. After performing our permutation test, we got a p value of 0.986, and since our p value is much higher than 0.05 we fail to reject the null hypothesis as there is no evidence that the model performs worse against healthy recipes. Based on our results, we can conclude that the model actually performs better for healthy recipes, opposed to performing better on unhealthy recipes.


<iframe src="assets/fairness_histogram.html" width="100%" height="500"></iframe>






