# 🧠 Recipe for Success: Predicting What Makes Food.com Recipes 5-Star Hits

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