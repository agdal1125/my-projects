---
layout: post
title: Cocktail Recipe Analysis (part.1)
date: 2019-10-25 
description: Collecting Cocktail Recipe Data
img: cocktail_1.jpg
---
## Cocktail Recipe Analysis  (part. 1)

### Initiatives

Drinking alcohol is a great way of breaking ice, networking with people. As a person who has a fairly low tolerance on alcohol, I prefer drinking cocktails over other types of drinks. They look fancy, come in many different flavors, and contains enough alcohol to let my shy personality go off for a while. 

However cocktails also give me a big challenge when it comes to choosing the right one...

<br>

<img src="https://media3.giphy.com/media/5WkqT5t0V3DCAeBsju/giphy.gif" width="600" height="300">

<br>

There are lots of cocktail recipe in this world and everytime I try a new bar only a few names are recognizable: mojito, old-fashioned, sex on the beach, cosmopolitan. Also when it comes to making a decision, I am like a mild version of Chidi from "the Good Place". Hence, it is very difficult to choose which cocktail I should drink! On top of that some bars just name their cocktails in a different way, which makes me regret my decision.

So I initiated this project: I will perform a thorough analysis on cocktail recipes to help my poor decision making skills and people who can relate to my situation. 

<br>

### Possible Solutions

There are a several possible approaches to this problem:

1. Recommendation System

   > Based on the information on the menu, I can develop a recommendation system to make an optimal decision.

2. Ingredient Network Analysis

   > By analyzing ingredients in recipes, I can obtain information about which cocktail ingredients I like and dislike.

3. Don't drink cocktails

   > This is not an option for me.

&nbsp;

Before choosing strategy, I should look at the data first. So, let's collect some data!

<br>

### Data Collection

Numerous pages have information about cocktail recipes, but I believe [this website](https://drinksmixer.com) has the most extensive recipes. Let's get our data!

<br>

**Collecting links to individual cocktail recipes**:

```python
# import packages
import requests
from bs4 import BeautifulSoup 
import time

base_url = "http://www.drinksmixer.com/cat/1/" 
pages = range(1,125)  # total 124 pages exist (2019.10.29)

cocktail_links = []
cocktail_name_list = []

for i in pages:
    
    # Set URL
    url = base_url+str(i)
    req = requests.get(url)
    html = req.text
    
    # Parse HTML with bs4
    soup = BeautifulSoup(html,'html.parser')
    
    # Find all recipe links
    drinks_box = soup.find("div",{"class":"m1"}).find("div",{"class":"min"}).find("div",{"class":"clr"}).find("tr")
    urls_in_page = drinks_box.find_all("a")
    
    # Loop through the pages to get all the information links of cocktails
    for link in urls_in_page:    
        cocktail_links.append("http://www.drinksmixer.com" + link["href"])
        cocktail_name_list.append(link.text)
        
# Check when you collected your data
print("Links collected in {}".format(time.ctime()))
```

> Links collected in Fri Oct 25 15:28:23 2019

```python
print(len(cocktail_links),len(cocktail_name_list))
```

> 12334 12334

So, we have total of 12334 recipes. Wow! I was overwhelmed with the number of results. I can say that justifies the whole point of this project. Now let's get the ingredients!

<br>

**Collecting cocktail recipes**

```python
from selenium import webdriver

# Use your path to locate your chromedriver
path_to_chromedriver = "/Users/nowgeun/Desktop/chromedriver"
driver = webdriver.Chrome(path_to_chromedriver)

# List to track progress
done = []

# Dictionary to save our results
cocktail_recipes = {}

# Loop through the 12334 links we have collectee
for one_url in cocktail_links:
    driver.get(one_url)
    
    # Cocktail name
    cocktail_name = driver.find_element_by_class_name("recipe_title").text

    # Cocktail Recipe
    cocktail_recipe = driver.find_element_by_class_name("recipe_data").find_elements_by_class_name("ingredient")
    
    recipe_dict = {} # Recipe of one cocktail
    
    for ingrdnt in cocktail_recipe:
        amount = ingrdnt.find_element_by_class_name("amount").text
        ing_name = ingrdnt.find_element_by_class_name("name").text

        recipe_dict[ing_name] = amount
    
    # Save one cocktail recipe to the whole recipe dictionary
    cocktail_recipes[cocktail_name] = recipe_dict
    done.append(one_url)
```

Let's check our collected data.

```python
 # The there should be equal number of cocktail names 
 len(cocktail_name_list) == len(cocktail_recipes.keys()) 
```

> False

Wait? Why is `False` returned here?? I assumed that some of the recipes were redundant and might have been repeated among the `cocktail_name_list`. 

```python
# Redundant Recipes were in the list
assert len(set(cocktail_name_list)) == len(cocktail_recipes.keys())
print(len(cocktail_recipes.keys()))
```

> 12242

Booyah! That's what I thought. So we have 12242 unique recipes in our dataset. 

Before continuing, let's save the data first. 

<br>

### Saving Data using pickle

```python
import pickle

with open("./pickle_data/list_of_cocktail_recipe_links.pickle", "wb") as a:
    pickle.dump(cocktail_links, a)

with open("./pickle_data/list_of_cocktail_names.pickle", "wb") as b:
    pickle.dump(cocktail_name_list, b)
    
with open("./pickle_data/cocktail_recipe_dict.pickle", "wb") as c:
    pickle.dump(cocktail_recipes, c)
```

We saved our data. Let's double check whether the loaded data is consistent with what we have.

<br>

### Loading Data using pickle

```python
with open("./pickle_data/cocktail_recipe_dict.pickle", "rb") as j:
    cocktail_links2 = pickle.load(j)
    
with open("./pickle_data/cocktail_recipe_dict.pickle", "rb") as k:
    cocktail_name_list2 = pickle.load(k)
    
with open("./pickle_data/cocktail_recipe_dict.pickle", "rb") as l:
    cocktail_recipes2 = pickle.load(l)
    
assert cocktail_links == cocktail_links2
assert cocktail_name_list == cocktail_name_list2
assert cocktail_recipes == cocktail_recipes2

print("Everything is Fine")
```

> Everything is Fine

Great! Our loaded data is consistent to what we have collected. 



### Continuing on part 2....

Let's call it a day and resume the work later! In the next part, I will explore my data, inspect problems and clean them. 





















