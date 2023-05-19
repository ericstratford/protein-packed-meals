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

### Data Preparation

Before we start our data cleaning and analysis process, we're going to turn our *recipes* dataframe and *reviews* dataframe into one dataframe.

We start by merging the two datasets creating a new dataset of each recipe with the addition of that recipe's average rating

The rest of our analysis will be conducted with this new combined dataset.

### Data Cleaning

We start with the new dataframe where each row looks like this:

| name         |     id |   minutes |   contributor_id | submitted   | tags                                              | nutrition                            |   n_steps | steps                                             | description        | ingredients                              |   n_ingredients |   avg_rating |
|:-------------|-------:|----------:|-----------------:|:------------|:--------------------------------------------------|:-------------------------------------|----------:|:--------------------------------------------------|:-------------------|:-----------------------------------------|----------------:|-------------:|
| 2 point play | 290008 |         5 |           330545 | 2008-03-04  | ['15-minutes-or-less', 'time-to-make', 'course... | [65.5, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0] |         2 | ['pour ingredients into a cocktail shaker with... | courtesy dekuyper. | ['sour apple schnapps', 'absolut vodka'] |               2 |            5 |

Knowing that we want to explore the various meal types, we are going to start by working with 'tags' as it contains the tags we will use as meal classification. Because the values in 'tags' are strings, first had to turn each value in the column into a list which we can look through.

To take a deep dive into nutritional values, we need to do the same transformation to the 'nutrition' column. Then we are going to split up each nutritional value into its own column.

For this project, we are mostly going to be looking at the protein/calorie ratio, which measures the grams of protein per calorie in a given meal. To find this value, convert the protein PDV into grams and divide that by the calories. And because some recipes have 0 calories, we need to fix the statistic by replacing NaN values with 0.

Now that we can easily work with the Protein/Calorie statistic, shown as 'Pro/Cal', we need to get a column which classifies each recipe into its respective meal type.
For this project, the categories we will use are ['Breakfast', 'Lunch', 'Dessert', 'Snacks', 'Beverage', and 'Dinner']

After the meals are all classified, we will also remove all of the non-classified recipes.

We are going to finish by only keeping the columns we are interest in.

After the cleaning, we are left with a dataframe looking like this:

| meal      |   Pro/Cal |   Protein (g) |   Calories (#) |
|:----------|----------:|--------------:|---------------:|
| Dessert   | 0.0108382 |           1.5 |          138.4 |
| Dessert   | 0.0113856 |          10   |          878.3 |
| Dinner    | 0.0781877 |          19.5 |          249.4 |
| Breakfast | 0.0265215 |           9.5 |          358.2 |
| Breakfast | 0.0340492 |           6.5 |          190.9 |

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

## Assessment of Missingness

In much of the data we see in world, missing values are nothing out of the ordinary, and this dataset is no exception. In this section, we will explore these datasets' missing values and find out why they are missing.

We will be using the original dataset that we started with after merging the recipes and reviews.

For a quick refresh, the dataframe follows this format:

| name         |     id |   minutes |   contributor_id | submitted   | tags                                              | nutrition                            |   n_steps | steps                                             | description        | ingredients                              |   n_ingredients |   avg_rating |
|:-------------|-------:|----------:|-----------------:|:------------|:--------------------------------------------------|:-------------------------------------|----------:|:--------------------------------------------------|:-------------------|:-----------------------------------------|----------------:|-------------:|
| 2 point play | 290008 |         5 |           330545 | 2008-03-04  | ['15-minutes-or-less', 'time-to-make', 'course... | [65.5, 0.0, 0.0, 0.0, 0.0, 0.0, 0.0] |         2 | ['pour ingredients into a cocktail shaker with... | courtesy dekuyper. | ['sour apple schnapps', 'absolut vodka'] |               2 |            5 |

### NMAR Analysis

Firstly, we are going to figure out which of our columns contain missing values. With some simple code we find that the columns with missing values are *name*, *description*, and *avg_rating*.

Using the definition of NMAR (Not Missing at Random) as a missingness type which depends on the missing values themselves, it seems as though this dataset has no values which are NMAR. 
While it seems possible that *name* or *description* could be NMAR because they are provided by the recipe creator and some recipes are missing a description because they just can't be described, possible dependencies can still be surmised. For instance, some users might be more likely to not include descriptions. Or maybe the recipes that don't need descriptions are self-explanatory, in which there will be a dependency on 'n_steps' or 'n_ingredients'.

### Missingness Dependency

How about our other column with missing values *avg_rating*?

Let's plot the distribution of the missing values against other variables:

<iframe src="Assets/missingness-dist.html" width=800 height=600 frameBorder=0></iframe>

This graph shows the distribution of the Date Submitted (date when the recipe was uploaded) for recipes where *avg_rating* is missing along with recipes where *avg_rating* is not missing. While the distributions appear similar, it seems like recipes with missing ratings typically have a slightly later submitted date.

To see whether the missing ratings and non-missing ratings actually do come from different distributions, we are going to use a permutation test using the absolute difference in means test statistic.

After running the a permutation test with 500 samples, we get an observed difference of 273 days. Given the context, this number is quite substantial. The permutation test yields a p-value of 0.0, so at the 1% significance level, we are going to reject the null hypothesis that the distribution is the same for missing and non-missing ratings with respect to submitted dates.

We can better visualize the results of the permutation test in this graph:

<iframe src="Assets/missingness-results.html" width=800 height=600 frameBorder=0></iframe>

The results of this permutation test suggest that the missingness of ratings is dependent on the date submitted. This makes logical sense because newer recipes are less likely to have had the opportunity to be reviewed.


Now let's see if the missingness of rating is not dependent on any other values.

After performing the same data cleaning as we did earlier, we can extract the individual nutrition values into their own columns. For this test, we will use the *Sodium (PDV)* column.

Repeating the process that we did for the submitted dates, we get the following graph:

<iframe src="Assets/missingness-dist-2.html" width=800 height=600 frameBorder=0></iframe>

Due to the significant outliers in this graph, it's hard to really see what's going on, so let's perform the permutation test to see the differences.

<iframe src="Assets/missingness-results-2.html" width=800 height=600 frameBorder=0></iframe>

The observed absolute difference in means is 0.35044% and the p-value is 0.896. As a result, we fail to reject the null hypothesis that the missing ratings and non-missing ratings come from the same distribution. Thus, we can safely assume that the rating missingness is not dependent on the Sodium content of the pertaining recipe.

## Hypothesis Testing

Earlier during our exploratory data analysis, we observed that lunch appeared to be the meal which had the highest protein/calorie ratio, with an average of 4.9%. In this section, we're going to test whether or not this observation is due to chance.

First, let's outline our hypotheses:
- Null Hypothesis: Meal Type and the protein/calorie ratio are not related.
	- The observed difference in protein/calorie ratio is due to chance.
- Alternative Hypothesis: Meal Type and protein/calorie ratio are related.
	- Lunches have a higher protein/calorie ratio not because of chance.

For this test, we are going to use the average protein/calorie ratio as our test statistic. Then we run a hypothesis test which creates 1000 sample averages generated from 5995 individual ratings and compares it to the observed average for lunch.

Our resulting p-value is 0.0, which tells us that we can reject the null hypothesis. Putting this visually:

<iframe src="Assets/hypothesis-test-results.html" width=800 height=600 frameBorder=0></iframe>

As we can see, the observed protein/calorie average for lunch of 0.049 does not appear to be due to chance, as confirmed by our hypothesis test.

From these findings, we can conclude (though not absolutely) that lunch is the meal of the day with the highest protein per calorie ratio when looking at the recipes on food.com.