# Exploring the Seasonal Trends in Recipe Ratings

## Introduction
Food is a fundamental aspect of human life, essential for survival. While some people eat food simply out of necessity, others savor the experience of trying new tastes and combining flavors to create something extraordinary. Regardless of what ones relationship with food is, exploring and analyzing different aspects of what makes recipes great is necessary. People often rely on ratings to choose foods and recipes, but the accuracy of these reviews can be questioned due to external factors affecting the reviewers' mood. One such factor is the season, as a gloomy, cold climate may evoke more negative emotions comparted to a warm, sunny climate. Thus, I posed the question of how the time of the year a recipe is posted affects their given rating. To investigate the relationship between seasons and ratings, I analyzed the two datasets consisting of recipe and ratings.

The first dataset, `recipes`, consists of 83782 rows and 12 columns with the following information:
| Columns          | Description                    |
| ---------------- | ------------------------------ |
| `name`           | Recipe name                    | 
| `id`             | Recipe ID                      |
| `minutes`        | Minutes to make recipe         |
| `contributor_id` | User ID who submitted recipe   |
| `submitted`      | Date recipe was submitted      |
| `tags`           | Tags for recipe from Food.com  |
| `nutrition`      | Information in the form of [number of calories, total fat (PDV), sugar (PDV), sodium (PDV), protein (PDV), saturated fat (PDV), carbohydrates (PDV)] where PDV is percentage of daily value |
| `n_steps`        | Number of steps                |
| `steps`          | In order text for recipe steps |
| `description`    | User-provided description      |

## Data Cleaning and Exploratory Data Analysis

<iframe
  src="assets/recipe_vs_rating-scatter-plot.html"
  width="800"
  height="600"
  frameborder="0"
></iframe>

## Assessment of Missingness

## Hypothesis Testing

## Framing a Prediction Problem

## Baseline Model

## Final Model 

## Fairness Analysis
