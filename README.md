# Recipe Insights: Healthiness, Ratings, and Recipe Complexity

Authors: Gabriel Borjas-Perez & Jared Parra

## Introduction

If you’ve ever searched for a dinner idea online, you know how overwhelming it can be! Thousands of recipes, each claiming to be “easy,” “healthy,” or “five-star.” What’s much less obvious is how people actually respond to those recipes once they cook them. Do home cooks reward lighter, healthier dishes with high ratings, or do the richest, most indulgent recipes still win?

In this project, we investigate how the **healthiness** and **complexity** of recipes relate to how highly they are rated on Food.com. Our central question is:

> **How are recipe ratings related to both their nutritional profile (healthy vs. unhealthy) and their complexity (number of steps)?**

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






