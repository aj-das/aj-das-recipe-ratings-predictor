# Recipe for Success: Predicting What Makes Food.com Recipes 5-Star Hits

**Author**: AJ Das  
**Email**: arinjoy@umich.edu  

This project explores which recipe features are most associated with high ratings using the Recipes and Ratings dataset. The analysis includes exploratory data visualization, feature engineering, and a predictive modeling pipeline.

---

## Introduction

This project uses the **Recipes and Ratings** dataset from Food.com, a large-scale collection of user-submitted recipes and their corresponding reviews. The dataset consists of over 230,000 reviews and 230,000+ recipes posted since 2008, making it an ideal source for exploring trends in food preferences and cooking behavior.

The main question I want to explore is: **What makes a recipe well-rated on Food.com?**  

I aim to identify which recipe features like preparation time, nutritional content, or complexity, are most associated with high user ratings. Ultimately, I want to predict the average rating of a new recipe before it receives any user feedback.

Understanding which features contribute to higher-rated recipes can help cooks tailor their recipes for success, start a recommendation systems on cooking platforms, and reveal insights about user preferences. 

This analysis has practical value both for individual users and platforms aiming to improve recipe suggestions or highlight high-potential meals.

### Dataset Overview  

After merging the original datasets, I worked with a cleaned dataset containing approximately **230,000 rows**.

Here are the key columns used in my analysis and modeling:

| Column           | Description                                                                 |
|------------------|-----------------------------------------------------------------------------|
| `minutes`        | Total time required to prepare the recipe                                   |
| `tags`           | Descriptive tags such as "easy", "vegan", or "holiday"                      |
| `nutrition`      | A list containing [calories, total fat, sugar, sodium, protein, saturated fat, carbohydrates] as percent daily values |
| `n_steps`        | Number of steps in the recipe                                                |
| `avg_rating`     | The average user rating for the recipe, computed from all submitted reviews |
| `description`    | The user-submitted text description of the recipe                           |

This combination of numeric and textual metadata provides insight for predicting a recipe’s potential success.

---

## Data Cleaning and Exploratory Data Analysis

### Data Cleaning

The raw dataset was comprised of receipe metadata and user-submitted reviews. I performed a **left merge** on the recipe ID to consolidate all of the information. After the merge, I cleaned and transformed the data by removing zero ratings. If a rating had a value of 0, I replaced it with `NaN`, since they indicated an absence of a rating rather than a low rating. I computed `avg_rating`, which represented the average user rating for each recipe. This become the target variable for prediction. The nutrition column was originally a stringified list. I parsed it into individual numeric columsn for `calories`, `total_fat`, `sugar`, `sodium`, `protein`, `saturated_fat`, and `carbohydrates`. Finally, reciepes without an average rating were removed from the modeling set. Cleaning the data helped structure to analyze and model for the data analysis.

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

Most recipes tend to be rated quite highly, with a strong right skew toward 5-star ratings. This bias in user feedback is important to consider when interpreting model performance.

#### Distribution of Calories
<iframe src="assets/calories_dist.html" width="800" height="500" frameborder="0"></iframe>

Calories are heavily right-skewed, with most recipes under 1,000 calories but a few extreme outliers reaching up to 40,000+. We retained these for modeling but scaled the features appropriately.

### Bivariate Analysis

#### Minutes vs Average Rating
<iframe src="assets/mins_vs_avg_rating.html" width="800" height="500" frameborder="0"></iframe>

This scatter plot reveals a weak negative trend between cooking time (minutes) and average rating. Users slightly prefer quicker recipes, though high-rated dishes exist at all durations.

#### Steps vs Average Rating
<iframe src="assets/steps_vs_avg_rating.html" width="800" height="500" frameborder="0"></iframe>

This box plot shows that recipes with fewer steps generally earn higher ratings. However, recipes with more steps tend to have more variation, potentially reflecting the challenge (and reward) of complex dishes.

### Interesting Aggregates

I created a new column called `is_easy`, which flags whether a recipe contains the tag "easy". Knowing if a recipe is easy may influence user perception and whether simplicity correlates with better ratings. 

I then grouped the data by this flag and calculated the mean average rating for each group:

| is_easy | avg_rating |
|---------|------------|
| False   | 4.67       |
| True    | 4.68       |

### Imputation
Several features had missing values, mainly the parsed nutrition columns. For modeling, we used median imputation via SimpleImputer in a pipeline to handle missing data.

#### Protein Distribution Before and After Imputation
<iframe src="assets/protein_before.html" width="800" height="400" frameborder="0"></iframe> 
<iframe src="assets/protein_after.html" width="800" height="400" frameborder="0"></iframe>

The overall shape of the distribution was perserved by filling in missing protein values with the median. This helped stabilize model performance without dropping rows and not introducing outliers.

## Framing a Prediction Problem

The goal of this project is to **predict the average user rating (`avg_rating`) of a recipe** based on its metadata. Since `avg_rating` is a **continuous numerical value ranging from 1 to 5**, this is a **regression problem**. I want to estimate a real-valued prediction of how well a recipe is expected to be rated, prior to any user reviews being submitted. The choice of `avg_rating` as the response variable aligns directly with our research question: *What makes a recipe well-rated?*

Understanding how different recipe characteristics influence user satisfaction can inform recommender systems and help both users and platforms highlight high-quality content.

To evaluate the model, I chose **Mean Squared Error (MSE)** as my metric. MSE penalizes larger errors more severely, encouraging models that are accurate across the board. Classification metrics like accuracy or F1-score would not be appropriate in this context because we are predicting a continuous outcome, not a class label.

At the **time of prediction**, all features used (e.g., `minutes`, `protein`, `n_steps`, `tags`, etc.) would be available to the system before the user submits any rating. Therefore, my setup realistically simulates how a platform like Food.com might recommend or rank recipes before they are reviewed.