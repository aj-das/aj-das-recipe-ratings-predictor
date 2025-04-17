By AJ Das, arinjoy@umich.edu  

## Introduction

I used the **Recipes and Ratings** dataset from Food.com to explore the question, **What makes a recipe well-rated on Food.com?**

The dataset contains 230,000+ recipes and reviews posted since 2008. I aim to identify which recipe features are most associated with high user ratings, such as preparation time, nutritional content, or complexity. Ultimately, I want to predict a new recipe's average rating before it receives any user feedback.

Understanding which features contribute to higher-rated recipes can help cooks tailor their recipes for success. It also reveals insight into the user's preferences and establishes recommendation systems on cooking platforms.

### Dataset Overview  

| Column           | Description                                                                 |
|------------------|-----------------------------------------------------------------------------|
| `minutes`        | Total time required to prepare the recipe                                   |
| `tags`           | Descriptive tags such as "easy", "vegan", or "holiday"                      |
| `nutrition`      | A list containing [calories, total fat, sugar, sodium, protein, saturated fat, carbohydrates] as percent daily values |
| `n_steps`        | Number of steps in the recipe                                                |
| `avg_rating`     | The average user rating for the recipe, computed from all submitted reviews |
| `description`    | The user-submitted text description of the recipe                           |

This metadata provides perception for predicting a recipe’s potential success.

---

## Data Cleaning and Exploratory Data Analysis

### Data Cleaning

The raw dataset was comprised of recipe metadata and user-submitted reviews. I performed a **left merge** on the recipe ID to consolidate all the information. After the merge, I cleaned and transformed the data by removing zero ratings. If a rating had a value of 0, I replaced it with `NaN`, since they indicated an absence of a rating rather than a low rating. For each recipe, I computed `avg_rating`, the target variable for prediction which represented the average user rating. Initially, the nutrition column was a stringified list so I parsed it into an individual numeric column for `calories`, `total_fat`, `sugar`, `sodium`, `protein`, `saturated_fat`, and `carbohydrates`. Finally, recipes without an average rating were removed from the modeling set. Cleaning the data helped structure and model for the data analysis.

| calories | total_fat | sugar | sodium | protein | saturated_fat | carbohydrates | avg_rating |
|----------|-----------|-------|--------|---------|----------------|----------------|-------------|
| 138.4    | 10.0      | 50.0  | 3.0    | 3.0     | 19.0           | 6.0            | 4.0         |
| 595.1    | 46.0      | 211.0 | 22.0   | 13.0    | 51.0           | 26.0           | 5.0         |
| 194.8    | 20.0      | 6.0   | 32.0   | 22.0    | 36.0           | 3.0            | 5.0         |
| 194.8    | 20.0      | 6.0   | 32.0   | 22.0    | 36.0           | 3.0            | 5.0         |
| 194.8    | 20.0      | 6.0   | 32.0   | 22.0    | 36.0           | 3.0            | 5.0         |

### Univariate Analysis

#### Distribution of Average Ratings
<iframe src="assets/avg_rating_dist.html" width="800" height="500" frameborder="0"></iframe>

Most recipes are rated quite highly, with a strong right skew toward 5-star ratings. It's important to understand bias in user feedback when interpreting model performance.

#### Distribution of Calories
<iframe src="assets/calories_dist.html" width="800" height="500" frameborder="0"></iframe>

Calories are heavily right-skewed, with most recipes under 1,000 calories but a few extreme outliers reaching up to 40,000+. I scaled the features appropriately.

### Bivariate Analysis

#### Minutes vs Average Rating
<iframe src="assets/mins_vs_avg_rating.html" width="800" height="500" frameborder="0"></iframe>

This scatter plot shows a weak negative trend between cooking time (minutes) and average rating. Users slightly prefer quicker recipes, though high-rated dishes exist at all durations.

#### Steps vs Average Rating
<iframe src="assets/steps_vs_avg_rating.html" width="800" height="500" frameborder="0"></iframe>

This box plot shows that recipes with fewer steps generally earn higher ratings. However, recipes with more steps tend to have more variation, which may reflect the challenge of complex dishes.

### Interesting Aggregates

I created a new column called `is_easy`, which flags whether a recipe contains the tag "easy". If a recipe is easy to make, then the user is more willing to try it.

I grouped the data by this flag and calculated the mean average rating for each group:

| is_easy | avg_rating |
|---------|------------|
| False   | 4.67       |
| True    | 4.68       |

### Imputation

The parsed nutrition columns had missing values, so I used median imputation via `SimpleImputer` to handle missing data. 

#### Protein Distribution Before and After Imputation
<iframe src="assets/protein_before.html" width="800" height="400" frameborder="0"></iframe> 
<iframe src="assets/protein_after.html" width="800" height="400" frameborder="0"></iframe>

The overall shape of the distribution was preserved by filling in missing protein values with the median. This stabilized the model performance without dropping rows and not introducing outliers.

## Framing a Prediction Problem

The goal of this project was to **predict the average user rating (`avg_rating`) of a recipe**. Since `avg_rating` is a continuous numerical value ranging from 1 to 5, this is a regression problem. I wanted to estimate a real-valued prediction of how well a recipe is expected to be rated before any user reviews are submitted. The choice of `avg_rating` as the response variable aligns directly with my research question: *What makes a recipe well-rated?*

To evaluate the model, I chose Mean Squared Error (MSE) as my metric. MSE penalizes substantial errors more severely. Classification metrics like accuracy or F1-score would not be appropriate in this context because we are predicting a continuous outcome, not a class label.

At the time of prediction, I used all features (e.g., `minutes`, `protein`, `n_steps`, `tags`, etc.) that were available to the system before the user submitted any rating. Therefore, my setup simulates how a platform like Food.com might recommend or rank recipes before they are reviewed.

## Baseline Model

I trained a linear regression model to have a starting point for predicting a recipe's average user rating. I used `minutes` (prep time) and `calories` (energy content) as inputs. These features may affect user perception and satisfaction, like whether the recipe takes a long or short time to make or considering the amount of calories in a recipe.

Before training, I dropped any rows where the `avg_rating` was missing. The preprocessing pipeline consisted of imputation, scaling, and modeling. Any missing values in features were filled with the median. I used `StandardScaler()` to improve numerical stability. Finally, I trained the model with `LinearRegression` from `sklearn`. I bundled the process into a `Pipeline` and evaluated the model using **Mean Squared Error (MSE)** on a 20% test set.

**Baseline MSE:** `0.2400`

Though this model is simple, it's a rudimentary standard to compare against advanced models with different feature engineering.

## Final Model

I introduced three new features and used a stronger modeling algorithm for my final model. I added:
- `protein_per_min`, which represents nutritional efficiency (protein content divided by preparation time)
- `is_easy`, a boolean value that determines whether the recipe is easy or not
- `n_steps` is the number of steps in the recipe

I chose a `RandomForestRegressor` to capture nonlinear relationships and interactions between these features. I applied `GridSearchCV` to evaluate combinations of the following hyperparameters:
- `n_estimators`: [50, 100]
- `max_depth`: [5, 10, None]

I utilized a median imputation strategy for numeric columns, applied standard scaling, and trained the model on 80% of the data using a pipeline structure. 

### Final Results
- Mean Squared Error (MSE): 0.1219
- Best Hyperparameters: `{'model__n_estimators': 100, 'model__max_depth': None}`

The MSE notably improved from the baseline model. Tuning feature engineering and hyperparameters helped capture complex trends in the data.

#### Feature Importances:
The bar chart shows the importance of each feature according to the Random Forest model.
<iframe src="assets/feature_importances.html" width="800" height="400" frameborder="0"></iframe>