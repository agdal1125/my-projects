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

<img src="/my-projects/assets/img/cocktail2_output_1.png" width="600" height="350">

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

<br>

### Identifying Potential Problems within the Data

While we took a look on our data, we discovered some problems and challenges:

1. **Too many units **

   >  oz, ml, tbsp, tsp, etc... we want the all units to be consistent, if possible.

2. **Dealing with variation of cocktails**

   > Zombie #1, Zombie #2, etc... variation of cocktails might influence the analysis by putting weights on specific ingredients.

3. **Sparse matrix**

   > Our dataset is 1974 x 12242 matrix, and it is sparse

4. **Same ingredients are in different names**

   > Some of the ingredients appear on different names in the recipe. Lemon juice, drop of lemon and  pressed lemon are the same thing. We need to find a way to unify the synonyms.



The top priority here is unifying the measurement units. We have no other choice but to do this in a hardcore way... 

<br>

### Data Cleansing: Working with Units

Let's start with the tractable problem first, and deal with the tougher ones later. Using regular expression, we can find  properly spaced patterns. The most desirable format would be a combination of number and the unit of amount. (or also a space in between them). For example, "1 oz", "4 ml", "5tbsp". I chose fluid ounce (*oz*) as the unified measurement as we all are used to the concept of 'shots' which is in general 1 *oz*. In order to convert all the units to *oz*, I created a dictionary to use it for conversion:

```python
# use oz as standard unit
liquid_units = {"oz":1,
                "ml":0.033814,
                "cl": 0.33814,
                "tsp":0.166667,
                "teaspoon":0.166667,
                "tea spoon":0.166667,
                "tbsp":0.5,
                "tablespoon":0.5,
                "table spoon":0.5,
                "cup": 8,
                "cups": 8,
                "qt":0.03125,
                "quart":0.03125,
                "drop":0.0016907
               }
```

<br>

Another problem I encountered with the recipes was that the way of presenting rational numbers were inconsistent as well. Some recipes used fractions such as 1/3 glass of apple juice, which makes more intuition in our real life than saying 0.3333 glass of apple juice. These fractions also had to be converted to floats for computational purpose. Hence, I made a function to convert fraction to floats:

```python
def frac_to_dec_converter(num_strings):
    """
    Takes a list of strings that contains fractions and convert them into floats.

    @Params
    - list_of_texts: list of str

    @Returns
    - list of floats

    @Example:
    [ln] >> frac_to_dec_converter(["1", "1/2", "3/2"])
    [Out] >> [1.0, 0.5, 1.5]
    """
    result = []

    for frac_str in num_strings:
        try:
            converted = float(frac_str)
        except ValueError:
            num, denom = frac_str.split('/')
            try:
                leading, num = num.split(' ')
                total = float(leading)
            except ValueError:
                total = 0
            frac = float(num) / float(denom)
            converted = total + frac

        result.append(converted)
        
    return result
```

<br>

Finally, I made a converting function to unify different units in the ingredients to our standard unit *oz*. The function was structured to convert only the units introduced in the dictionary that I made in the previous code chunk. This function has several limitations:

1. It does not resolve co-reference problem 

   > There are several different ways to call a single measurement unit. For example, *cl* can be named as centi-litres, centi-liter or even cases when it has been mis-spelled. 

2. It only converts units stated in the dictionary

   > There may be other units that I may have not captured while exploring through my data. Those units will be left intact.

<br>

Using the functions created above, our data can be processed:

```python
# converting units within each column
for drink in df.columns:
    df[drink] = unit_unify(df[drink])

# saving the dataframe into a new file
df.to_csv("recipe_cleaned_v1.csv")
```

<br>

We dealt with the problem of recipe data having too many units. The other problems will be addressed in the upcoming posts!