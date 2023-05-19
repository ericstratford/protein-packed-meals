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

We start with the recipes dataframe where each row looks like this:

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

We are going to finish by only keeping the columns we are interest in
```py
recipes = recipes[['id','meal','Pro/Cal','Protein (g)','Calories (#)']].set_index('id')
```

After the cleaning, we are left with a dataframe looking like this:

| meal      |   Pro/Cal |   Protein (g) |   Calories (#) | Pro/Cal bin         |
|:----------|----------:|--------------:|---------------:|:--------------------|
| Dessert   | 0.0108382 |           1.5 |          138.4 | (-0.000235, 0.0118] |
| Dessert   | 0.0113856 |          10   |          878.3 | (-0.000235, 0.0118] |
| Dinner    | 0.0781877 |          19.5 |          249.4 | (0.0705, 0.0823]    |
| Breakfast | 0.0265215 |           9.5 |          358.2 | (0.0235, 0.0353]    |
| Breakfast | 0.0340492 |           6.5 |          190.9 | (0.0235, 0.0353]    |

### Univariate Analysis

Now that our data is clean, we can start with our EDA (Exploratory Data Analysis).

To see the distribution and corrolations between our variabls, we can graph our data into histograms:

<iframe src="Assets/pcr-dist.html" width=800 height=600 frameBorder=0></iframe>

In the above graph, we can see the distribution of Pro/Cal (Protein/Calorie) ratios among the recipes we are looking at. The data is heavily skewed towards the left, showing that most recipes have between a 0% and 5% Pro/Cal ratio with a cluster at 0%. This suggests that we can expect our average Pro/Cal ratio to be around 1-3% among all meals.

<iframe src="Assets/meal-dist.html" width=800 height=600 frameBorder=0></iframe>

This graph displays the distribution of meal types, where the height of each bar represents the number of recipes in that classification. As shown here, most recipes are classified as dessert, dinner, lunch, breakfast, and beverage. 

Using information from the two graphs shown above, we can reasonably suppose that most of the lowest Pro/Cal values come from beverages and desserts, but let's take a deeper look to see the actual correlation between the variables.

### Bivariate Analysis

<iframe src="Assets/pcr-meal-box.html" width=800 height=600 frameBorder=0></iframe>

This box-plot shown above allows us to take a deeper look at the corrolation between meal types and their respective Pro/Cal ratio. While each meal type contains its high-protein outliers, we can see that 'Dessert' and 'Beverage' appears to have lower Pro/Cal ratios. At the same time, 'Dinner' and 'Lunch' seem to have the highest Pro/Cal ratios, with lunch have a slightly higher median.

### Interesting Aggregates

Let's get a simpler look at the Pro/Cal statistics of each meal type:

| meal      |      mean |     median |   count |
|:----------|----------:|-----------:|--------:|
| Beverage  | 0.0123483 | 0.00455685 |    4573 |
| Breakfast | 0.0353225 | 0.0302078  |    5147 |
| Brunch    | 0.0318117 | 0.0248217  |    1218 |
| Dessert   | 0.0141622 | 0.0122309  |   12915 |
| Dinner    | 0.0473276 | 0.0397944  |    6732 |
| Lunch     | 0.0494123 | 0.0442953  |    5996 |
| Snacks    | 0.0287974 | 0.0235849  |     669 |

Looking at the values in this table, we see that *Lunch* has both the highest mean and median. 

So that's it right? Lunch is the best meal for gym-bros?

Well, perhaps it's just the result of random chance that lunch has the most protein per calorie on average. We'll take a closer look at this in the *Hypothesis Testing* section.