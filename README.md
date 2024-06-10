# Exploring the Seasonal Trends in Recipe Ratings

## Introduction
Food is a fundamental aspect of human life, essential for survival. While some people eat food simply out of necessity, others savor the experience of trying new tastes and combining flavors to create something extraordinary. Regardless of what ones relationship with food is, exploring and analyzing different aspects of what makes recipes great is necessary. People often rely on ratings to choose foods and recipes, but the accuracy of these reviews can be questioned due to external factors affecting the reviewers' mood. One such factor is the season, as a gloomy, cold climate may evoke more negative emotions comparted to a warm, sunny climate. Thus, I posed the question of **how the time of the year a recipe is posted affects their given rating**. To investigate the relationship between seasons and ratings, I analyzed the two datasets consisting of recipe and ratings.

The first dataset, `recipes`, consists of *83782 rows* where each row is a recipe, and 12 columns with the following information:

| Columns         | Description                                                                                                                                  |
| --------------- | -------------------------------------------------------------------------------------------------------------------------------------------- |
| `name`            | Recipe name                                                                                                                                  |
| `id`              | Recipe ID                                                                                                                                    |
| `minutes`         | Minutes to make recipe                                                                                                                       |
| `contributor_id`  | User ID who submitted recipe                                                                                                                 |
| `submitted`       | Date recipe was submitted                                                                                                                    |
| `tags`            | Tags for recipe from Food.com                                                                                                                |
| `nutrition`       | Information in the form of [number of calories, total fat (PDV), sugar (PDV), sodium (PDV), protein (PDV), saturated fat (PDV), carbohydrates (PDV)] where PDV is percentage of daily value |
| `n_steps`         | Number of steps                                                                                                                              |
| `steps`           | In order text for recipe steps                                                                                                               |
| `description`     | User-provided description                                                                                                                    |
| `ingredients`     | List of ingredients required for the recipe                                                                                                  |
| `n_ingredients`   | Number of ingredients required for the recipe                                                                                               |

The second dataset, `interactions`, consists of *731927 rows* where each row is a review with a rating, and 5 columns with the following information:

| Columns     | Description          |
| ----------- | -------------------- |
| `user_id`   | User ID              |
| `recipe_id` | Recipe ID            |
| `date`      | Date rating given    |
| `rating`    | Rating given         |
| `review`    | Text of review       |

Since I am trying to determine whether there is a relationship between seasons and recipes ratings, I will focus on the columns `name`, `minutes`, `submitted`, and `nutrition` from the `recipes` dataset, and `date` and `rating` from the `interactions` dataset. For now, I will also need to include columns such as `id` from the `recipes` dataset and `recipe_id` from the `interactions` dataset so I will be able to combine the two datasets. However, before I can begin separating out the relevant columns, I need to clean the data. 

## Data Cleaning and Exploratory Data Analysis

### Data cleaning

To clean my data, I followed the pillars of data cleaning which are performing data quality checks, identifing and handling missing values, and performing transformations. However, before I began doing these steps, I merged the two datasets together into one dataset. 

1. I left merged the `recipes` and  `interactions` dataset by the `id` and `recipe_id` columns since they both represent recipe's IDs. We do not merge from the user ID columns because the same person who posted the recipe is probably not the same person who posted the review. Also, we want to keep multiple recipes with the same name so we perform a left merge. 

2. I filled all ratings of 0 with `np.nan` because ratings scale from 1 to 5 so a 0 rating is not possible. Also, ratings containing 0 values instead of simply being missing could inaccurately lower the average rating of a recipe.

3. Find the average rating per recipe and add to the dataframe as the `average_rating` column to better capture the recipe's real rating since recipes could contain multiple ratings. 

4. Next, I check the data types of the columns and converted columns that appeared like lists but were actually strings into actual lists. Some columns like `tags`, `steps`, and `ingredients` needed to be transformed into a list of strings, while a column like `nutrition` needed to be transformed into a list of floats.

5. I converted columns that contain dates, `submitted` and `date` into datetime data types so I can use them as Timestamp objects. I renamed `submitted` to `date_submitted` and `date` to `date_interacted` to better show that `submitted` is the date the recipe was submitted and `date` is the date the review/rating was posted or date someone interacted with the recipe. 

6. Next, I stripped the whitespaces from the `name` column.

7. I expanded the `nutrition` column into six new columns called `total_fat`, `sugar`, `sodium`, `protein`, `saturated_fat`, and `carbohydrates` to make it easier to explore every unique value in `nutrition `.

8. I added two columns that would better represent the season each review and recipe was posted. First, I added a column called `season_submitted` representing the season the recipe was submitted. Then, I added a column called `season_interacted` representing the season the review was submitted. Since, seasons are different depending on where in the world someone is located, I standardized the seasons to assume a North American climate due to the fact that the website the dataset was based off of, Food.com is American and this research project takes place in America. Thus, I labeled the seasons winter if the year of `date_submitted` or `date_interacted` was December through Feburary, spring if the year was March through May, summer if the year was June through August, fall if the year was  September through November.

9. I added a more general column called `season_category` to demonstrate the general climate of the different seasons of when the recipe was posted or based on `season_submitted`. Again, for the reasons explained in step 8, I used the North American climate and labeled seasons in summer and spring as warm while seasons in winter and fall as cold.

10. I added another column called `days_between` to show the how long it took between when a recipe was first posted and when the review was posted. I filtered out recipes where a review was posted before the recipe itself because the rating may not accurately represent the current version of the recipe. Such reviews could be based on earlier, potentially different, recipes or a mistaken entry and thereby should not be included. I also filtered out rows where no dates were present.

11. Then, I added two columns `dayofweek_recipe` and `dayofweek_review` to represent the day of the week a recipe and its associated rating was posted. 

12. Finally, I created a clean version of my dataframe by including the columns `name`, `minutes`, `date_submitted`, `date_interacted`, `rating`, `average_rating`, `num_calories`, `total_fat`, `sugar`, `season_submitted`, `season_interacted`, `days_between`, `dayofweek_recipe`, and `dayofweek_review` as these were the columns that would help me answer my research question. 

Below is the head of the finalized dataframe, `df`:

| name                               |   minutes | date_submitted      | date_interacted     |   rating |   average_rating |   num_calories |   total_fat |   sugar | season_submitted   | season_category   | season_interacted   |   days_between | dayofweek_recipe   | dayofweek_review   |
|:-----------------------------------|----------:|:--------------------|:--------------------|---------:|-----------------:|---------------:|------------:|--------:|:-------------------|:------------------|:--------------------|---------------:|:-------------------|:-------------------|
| 1 brownies in the world best ever  |        40 | 2008-10-27 00:00:00 | 2008-11-19 00:00:00 |        4 |                4 |          138.4 |          10 |      50 | fall               | cold              | fall                |             23 | Mon                | Wed                |
| 1 in canada chocolate chip cookies |        45 | 2011-04-11 00:00:00 | 2012-01-26 00:00:00 |        5 |                5 |          595.1 |          46 |     211 | spring             | warm              | winter              |            290 | Mon                | Thurs              |
| 412 broccoli casserole             |        40 | 2008-05-30 00:00:00 | 2008-12-31 00:00:00 |        5 |                5 |          194.8 |          20 |       6 | spring             | warm              | winter              |            215 | Fri                | Wed                |
| 412 broccoli casserole             |        40 | 2008-05-30 00:00:00 | 2009-04-13 00:00:00 |        5 |                5 |          194.8 |          20 |       6 | spring             | warm              | spring              |            318 | Fri                | Mon                |
| 412 broccoli casserole             |        40 | 2008-05-30 00:00:00 | 2013-08-02 00:00:00 |        5 |                5 |          194.8 |          20 |       6 | spring             | warm              | summer              |           1890 | Fri                | Fri                |

### Univariate Analysis


### Bivariate Analysis

<iframe
  src="assets/recipe_vs_rating-scatter-plot.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

### Interesting Aggregates

## Assessment of Missingness

## Hypothesis Testing

## Framing a Prediction Problem

## Baseline Model

## Final Model 

## Fairness Analysis
