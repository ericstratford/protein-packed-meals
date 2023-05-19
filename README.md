# The Gym-Bro's Favoriate Meal of the Day

by Eric Stratford (estratford@uscd.edu)

Which meal type has the most protein per calorie? Using data analysis and statistical testing, we will explore this question in this project for DSC 80 at UCSD.

---

## Table of Contents
{: .no_tox .text-delta }


---

## Introduction

Let's say you're a gym-bro who wants to cut down on the number of meals you eat per day. Which meal of day should you cut out and which should you keep? 
In this project, we're going to explore the question:

**"Which type of meal packs the most protein per calorie on average?"**

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

Let's start analyzing the data.

---

## Cleaning and Exploratory Data Analysis (EDA)

### Data Cleaning

Starting with the recipes dataframe where each row looks like this:

| name         |     id |   minutes |   contributor_id | submitted   | tags                                              | nutrition                            |   n_steps | steps                                             | description        | ingredients                              |   n_ingredients |
|:-------------|-------:|----------:|-----------------:|:------------|:--------------------------------------------------|:-------------------------------------|----------:|:--------------------------------------------------|:-------------------|:-----------------------------------------|----------------:|
| 2 point play | 290008 |         5 |           330545 | 2008-03-04  | ['15-minutes-or-less', 'time-to-make', 'course... | [65.5, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0] |         2 | ['pour ingredients into a cocktail shaker with... | courtesy dekuyper. | ['sour apple schnapps', 'absolut vodka'] |               2 |

Knowing that we want to explore the various meal types, we are going to start by working with 'tags' as it contains the tags we will use as meal classification. Because the values in 'tags' are strings, we're going to use the code:
```py
recipes['tags'] = recipes['tags'].str.strip("[]").str.replace("'","").str.split(", ")
```
to turn each value in the column into a list which we can look through.
Because we are also going to take a deep dive into nutritional values, we need to do the same transformation to the 'nutrition' column. Then we are going to split up each nutritional value into its own column:
```py
recipes['nutrition'] = recipes['nutrition'].str.strip("[]").str.split(",")
nutrition_labels = ["Calories (#)", "Total Fat (PDV)", "Sugar (PDV)", "Sodium (PDV)", "Protein (PDV)", "Saturated Fat (PDV)", "Carbohydrates (PDV)"]
nutrition_df = pd.DataFrame(recipes['nutrition'].apply(pd.Series).astype(float))
nutrition_df.columns = nutrition_labels
recipes = pd.concat([recipes, nutrition_df], axis=1).drop(columns=['nutrition'])
```
For this project, we are mostly going to be looking at the protein/calorie ratio, which measures the grams of protein per calorie in a given meal. To find this value, we will use the code:
```py
recipes['Protein (g)'] = recipes['Protein (PDV)']/2
recipes['Pro/Cal'] = recipes['Protein (g)']/recipes['Calories (#)']
```
And because some recipes have 0 calories, we need to fix the statistic with
``` py
recipes['Pro/Cal'] = recipes['Pro/Cal'].fillna(0)
```

Now that we can easily work with the Protein/Calorie statistic, shown as 'Pro/Cal', we need to get a column which classifies each recipe into its respective meal type.
For this project, the categories we will use are ['Breakfast', 'Lunch', 'Dessert', 'Snacks', 'Beverage', and 'Dinner']
We create the classification and only keep the relevant recipes with
```py
meals = ['breakfast', 'lunch', 'desserts', 'snacks', 'beverages', 'brunch', 'dinner-party']
meal_dict = {'breakfast':'Breakfast', 'lunch':'Lunch', 'desserts':'Dessert', 'snacks':'Snacks',\
             'beverages':'Beverage', 'brunch':'Brunch', 'dinner-party':'Dinner'}
recipes['meal'] = recipes['tags'].apply(lambda tags: next((tag for tag in tags if tag in meals), np.nan))
recipes['meal'] = recipes['meal'].replace(meal_dict)
recipes = recipes[recipes['meal'].isna()==False]
```

And we are going to finish by only keeping the columns we are interest in
```py
recipes = recipes[['id','meal','Pro/Cal','Protein (g)','Calories (#)']].set_index('id')
```
