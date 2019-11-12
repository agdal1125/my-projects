---
layout: post
title: Cocktail Recipe Analysis (part.2)
date: 2019-11-04 
description: Exploring Cocktail Recipe Data
img: cocktail_2.jpg
tags: [Cockail, EDA, Data Processing, Python] # add tag
---

Now that we have our dataset, it's time for exploration!
Let's load the data and clean them first.

<br>

### Load Data

```python
import pickle
import pandas as pd
import numpy as np

with open("./pickle_data/cocktail_recipe_dict.pickle", "rb") as g:
    cocktail_recipes = pickle.load(g)
```

Remember, `cocktail_recipes` is a dictionary object. 

<br>

### Cleaning Data

The keys in `cocktail_recipes` are names of cocktail recipes. It would be annoying to keep the string " recipe" here. Let's delete it.

```python
# New dictionary to store our recipes
data_dict = {}

# Loop through the dictionary and delete string " recipe" in the key
for key,value in cocktail_recipes.items():
    data_dict[key.replace(" recipe","")] = value
 
# Convert to pandas dataframe
df = pd.DataFrame(data_dict)
df = df.fillna("") # fill NaN value with empty string ""
print(df.shape)
df.head()
```

<img src="/assets/img/cocktail2_output_1.png" width="600" height="350">

<br>

### Exploratory Data Analysis

Now that we have our dataset, let's explore and find out how to wrangle this

##### Some tests & checks

```python
# Test that no column names overlap due to upper or lower case
assert len(list(map(lambda x: x.lower(), list(df.columns)))) == len(set(df.columns))
```

```python
# How many different values are existent?
all_values = list(np.unique(df.values))
len(all_values)
```

> 1713

```python
# Let's take a look at the measurements of ingredients
from random import randint

### Generate a list of random numbers
rand_num_list = [randint(0,1713) for i in range(0,20)]

### Let's see some sample...
[all_values[i] for i in rand_num_list]
```

> ```
> ['16 parts',
>  'float',
>  '2 ozfresh ruby red',
>  '15 oz',
>  '5 ozfrench',
>  '1 halved, canned',
>  '1 ozcold black',
>  '5 crushed',
>  '5 - 6 ozice cold',
>  '1/3 liter',
>  'Guava',
>  '4 ozlukewarm',
>  '1 1/5 cups',
>  '1 splashcold',
>  '1/3 partfresh',
>  '4',
>  '4',
>  '3 ozGreen',
>  '3/4 ozpre-chilled',
>  '1/5']
> ```

Ok. So, we have 1713 different measurements and units. Some of the strings are not spaced well, some are just in numbers, units are different and some amounts are in ranges. Let's use regular expression to check how many unique number of unit measurements exist in our dataset.

<br>

```python
# Exploring all the units/measurement in this recipe

import re

no_numbers = [] # remove all numbers from the values
for amt in all_values:
    no_numbers.append(re.sub("[^a-zA-Z ]","",amt).strip())
    
no_numbers = list(set(no_numbers))
print(len(no_numbers))
print(no_numbers[:20])
```

> 835

> ```
> ['', 'ozgrated', 'ozchlled', 'squirtfresh', 'scoopsfrozen mango', 'canchilled', 'Pineapple', 'slicesfresh', 'several pureed', 'dropsfresh', 'jigger', 'ozPowdered', 'oz Olives', 'ozGrapefruit', 'a handful', 'mlchilled', 'ozlemon with', 'tbspfrozen', 'ozgold', 'tsppure'] 
> ```

835 different number of units exist here. Well let's search for the patterns. By context, we know that "ozgrated" is "oz grated" and "mlchilled" is "ml chilled". There are different ways to measure liquid: ounce(oz), milileter(ml), centileter(cl), tablespoon (tsbp), teaspoon (tsp), etc. Other than the listed measurements seems like units describing garnishes in cocktails. 

