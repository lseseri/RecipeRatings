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

I performed univariate analysis on the distribution of average ratings. 

<iframe
  src="assets/fig_rating-hist-plot.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>
In this histogram, we can see that the graph is left skewed with a majority of the average ratings being between 4 and 5. This suggests that our data from food.com either includes only highly rated recipes or people on food.com, a majority of the time, tend to rate recipes higher than lower. 

### Bivariate Analysis

I performed bivariate analysis on the relationship between the average rating of when a recipe was posted and the average rating of when a review was posted.

<iframe
  src="assets/recipe_vs_rating-scatter-plot.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>
<p style="margin-top: -20px;">Your text here...</p>
In this scatter plot, we can see from looking at the x-axis that the season the review was posted is overall higher for spring and summer than winter and fall so it seems that when people post ratings in spring and summer, they tend to be more generous and rate higher than when people review in winter and fall. Also, from looking at the legend, we can see that generally the green and yellow dots which refer to spring and summer are higher than the other dots which refer to winter and fall, thus it appears that the seasons when recipes were posted in spring and summer recieve higher ratings than those posted in winter and fall. 

### Interesting Aggregates

I made a pivot table by grouping by the season the recipe was posted and review was posted then finding the mean of the average rating for each of these categories. It demonstrates how recipes that were posted in spring and summer seem to have higher average ratings than those posted in winter and fall.

| season_interacted   |    fall |   spring |   summer |   winter |
|:--------------------|--------:|---------:|---------:|---------:|
| fall                | 4.66138 |  4.67628 |  4.67979 |  4.65328 |
| spring              | 4.68286 |  4.69551 |  4.68407 |  4.6554  |
| summer              | 4.67183 |  4.70875 |  4.69917 |  4.66939 |
| winter              | 4.67354 |  4.67018 |  4.6773  |  4.64726 |

I also grouped by the season category and found the mean to compare the differences between warm and cold seasons on how long it takes to prepare the recipes, the average rating of recipes, the number of calories of recipes, the total fat of recipes, the sugar of recipes, and the days between when a recipe was posted and a review was posted. What stood out to me was that in all these categories except the average rating, recipes posted in warmer seasons had a lower mean than colder seasons. For average ratings, recipes posted in warmer seasons, similar to what has been shown in the graphs, have a higher average rating than colder seasons. 

| season_category   |   minutes |   average_rating |   num_calories |   total_fat |   sugar |   days_between |
|:------------------|----------:|-----------------:|---------------:|------------:|--------:|---------------:|
| cold              |  125.488  |          4.66172 |        426.51  |     32.3507 | 64.7016 |        730.202 |
| warm              |   89.6232 |          4.69012 |        413.128 |     31.5245 | 63.0547 |        679.04  |

## Assessment of Missingness

In the merged dataset there are only three columns with missing values: `rating`, `review`, and `description`. 

### NMAR Analysis

-look up and see if need to change word review to rating-

A column in the dataset that I think is Not Missing At Random (NMAR) is the `review` column. Since people are typically more likely to submit a review and describe how they are feeling when they feel strongly about something such as when they are disappointed with a recipe or when they are pleased with a recipe, the fact that the review is missing could depend on the actual value missing. Thus, if a person was neutral about a recipe they will feel less inclined to spend their time leaving a description of how the recipe made them feel. Additional data I could obtain to explain the missingness and make it Missing At Random (MAR) is the `rating` column to see if the values that are missing are associated with the rating they left. Also, the time of day the review was posted could make it MAR. A rating is more likely to be missing if the time of day was late at night or early in the morning since the reviewer could be tired than if it was in the middle of the day. 

### Missingness Dependency

I decided to test if the missingness of `rating` depends on the `season_submitted` column, which is the season the recipe was submitted. For this permutation test, I used a significance level of 0.05 and test statistic of total variation distance (TVD) because I needed to measure the distance between two categorical distributions.

**Null Hypothesis: The distribution of the season when the recipe was submitted when rating is missing is the same as the distribution of the season when the recipe was submitted when the rating is not missing.**

**Alternative Hypothesis: The distribution of the season when the recipe was submitted when rating is missing is not the same as the distribution of the season when the recipe was submitted when the rating is not missing.**

<iframe
  src="assets/missingness-bar-plot.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

I ran the permuaation test by shuffling the `season_submitted` column 500 times to find different TVDs and compare that to my observed TVD of 0.039 which is show by the red line in the graph below. Since the p-value of 0.0 is less than the alpha 0f 0.05, I reject the null hypothesis. The distribution of seasons when a recipe was posted when a rating is missing is the same as the distribution when the rating is there. Thus, the permutation test does provide proof that the missingness of rating is dependent on the season the rating was submitting, making it MAR.  

<iframe
  src="assets/missingness-emp-dist.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

## Hypothesis Testing

I am curious on if the time of the year affects a recipe's average rating so I will be testing on whether the average rating is greater for warmer seasons consisting of summer and spring than colder seasons consisting of winter and fall. This investigation is significant because it will allow me to understand if seasonal trends do have an affect on the ratings of recipes. I will be using the columns `season_category` and `average_rating` for my test.

**Test Statistic:** difference in group means (warm - cold)

**Significance Level:** 0.05

**Null Hypothesis:** In the population, the average rating of recipes submitted in warmer seasons (summer and spring) and colder seasons (winter and fall) have the same distribution, and the observed differences in our samples are due to random chance.

**Alternative Hypothesis:** In the population, warmer seasons (summer and spring) have higher average ratings than colder seasons (winter and fall), and the observed difference in our samples cannot be explained by random chance alone.

I decided to do a permuation test since I have two groups, the warmer seasons and colder seasons, and I am trying to determine if they look like they were drawn from the same population. I generated new data by shuffling the group labels of warm and cold and computing the test statistic of difference in group means for each shuffle. I choose difference in group means because it allows me to measure how different the two numerical distributions are by doing the mean average rating for warmer seasons minus the mean average rating for colder seasons.

<iframe
  src="assets/hyp_test-emp-dist.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

Since my p-value of 0.0, as shown by the red line in the distribution above, is less than my significance level of 0.05, I reject the null hypothesis that the average rating of recipes submitted in warmer seasons and colder seasons have the same distribution. Thus, I can say that the difference between the two groups of warmer and colder seasons is statisically significant and that the average ratings of recipes that were submitted in warmer seasons are higher than those that were submitted in colder seasons. I can infer that there is a correlation between the time of year when recipes were submitted and the ratings they receive. However, it is important to note that while there is an association between the season a recipe was submitted and its rating, this does not necessarily imply that submitting a recipe during a specific season will cause it to receive a higher rating.

## Framing a Prediction Problem

## Baseline Model

## Final Model 

## Fairness Analysis
