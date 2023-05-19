# The Gym-Bro's Favorite Meal
Which meal type has the most protein per calorie? Using data analysis and statistical testing, we will explore this question in this project for DSC 80 at UCSD.

# The Gym-Bro's Favoriate Meal of the Day

by Eric Stratford (estratford@uscd.edu)

---

## Table of Contents
{: .no_tox .text-delta }

1. TOC
{:toc}

---

## Introduction

Let's say you're a gym-bro who wants to cut down on the number of meals you eat per day. Which meal of day should you cut out and which should you keep? 
In this project, we're going to explore the question:

**Which type of meal packs the most protein per calorie on average?**

To answer this question, we're going to be using a dataset of recipes from food.com. We are also going to be using a related dataset of reviews for each recipe for some preliminary data exploration.
The recipes dataset contains 83,782 rows, one for each recipe as well as the columns for recipe name, cooking instructions, ingredients, nutrition, etc. 
For the purpose of this question, we will be using the following relevant columns:
- "id": a unique identifier for each recipe
- "tags": a list of tags to classify each recipe (such as '60-minutes-or-less', 'lunch', 'thai')
- "nutrition": a list of values representing [calories, total fat, sugar, sodium, protein, saturated fat, carbohydrates] where calories is a number and the other nutritional values shown as percentage of daily value (PDV)

For the *missingness analysis*, we will also use the columns:
- "submitted": the date the recipe was posted
And the following columns from the review dataset:
- 'recipe_id': id of the recipe being reviewed
- 'rating': the rating given to the recipe by that review

Let's start analyzing this data.

---

## Cleaning and Exploratory Data Analysis (EDA)

### Data Cleaning

Starting with a dataframe that looks like this:

>| name                                 |     id |   minutes |   contributor_id | submitted   | tags                                                                                                                                                                                                                        | nutrition                                |   n_steps | steps                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             | description                                                                                                                                                                                                                                                          | ingredients                                                                                                                                                                    |   n_ingredients |
|:-------------------------------------|-------:|----------:|-----------------:|:------------|:----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|:-----------------------------------------|----------:|:----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|:---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|:-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------------:|
| 1 brownies in the world    best ever | 333281 |        40 |           985201 | 2008-10-27  | ['60-minutes-or-less', 'time-to-make', 'course', 'main-ingredient', 'preparation', 'for-large-groups', 'desserts', 'lunch', 'snacks', 'cookies-and-brownies', 'chocolate', 'bar-cookies', 'brownies', 'number-of-servings'] | [138.4, 10.0, 50.0, 3.0, 3.0, 19.0, 6.0] |        10 | ['heat the oven to 350f and arrange the rack in the middle', 'line an 8-by-8-inch glass baking dish with aluminum foil', 'combine chocolate and butter in a medium saucepan and cook over medium-low heat , stirring frequently , until evenly melted', 'remove from heat and let cool to room temperature', 'combine eggs , sugar , cocoa powder , vanilla extract , espresso , and salt in a large bowl and briefly stir until just evenly incorporated', 'add cooled chocolate and mix until uniform in color', 'add flour and stir until just incorporated', 'transfer batter to the prepared baking dish', 'bake until a tester inserted in the center of the brownies comes out clean , about 25 to 30 minutes', 'remove from the oven and cool completely before cutting'] | these are the most; chocolatey, moist, rich, dense, fudgy, delicious brownies that you'll ever make.....sereiously! there's no doubt that these will be your fav brownies ever for you can add things to them or make them plain.....either way they're pure heaven! | ['bittersweet chocolate', 'unsalted butter', 'eggs', 'granulated sugar', 'unsweetened cocoa powder', 'vanilla extract', 'brewed espresso', 'kosher salt', 'all-purpose flour'] |               9 |
