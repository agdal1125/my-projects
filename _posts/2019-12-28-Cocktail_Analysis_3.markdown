---
layout: post
title: Cocktail Recipe Analysis (part.3)
date: 2019-12-28
description: Developing & Constructing methodologies
img: cocktail_3.jpg
tags: [Recommendation System, Cocktail, ensemble, Recommender] 
---
In the previous post, we processed our data to standardize different liquid measurements into *oz*. Before processing our data, we must think of how to use of our data. We talked about the two ways of helping people to make decisions in the first post:

1. Recommendation System

2. Ingredient Network Analysis

Let's start with Recommendation System. 

<br>

## Ideas for Cocktail Recommendation System

There are several types of recommendation system depending on how it suggests or defines "relevant" items to people. Two major categories exists to define the "relevancy": **Collaborative Filtering Method** and **Content Based Method**. The diagram below is to facilitate illustration of the two methods.

<img src="https://www.researchgate.net/profile/Lionel_Ngoupeyou_Tondji/publication/323726564/figure/fig5/AS:631605009846299@1527597777415/Content-based-filtering-vs-Collaborative-filtering-Source.png">

<p style="font-size:7pt; text-align:center;"> source: : https://www.themarketingtechnologist.co/building-a-recommendation-engine-for-geeksetting-up-the-prerequisites-13/ </p>
Collaborative Filtering Method requires interaction data between users and items. In our case, it would be the rating data of cocktail by drinkers, which we do not have at this moment. To make use of our cocktail ingredient data, we should adopt Content Based Method; **we can recommend a similar cocktail to the ones that the drinker may have liked in the past, based on their ingredients**. Now, the problem we must solve is how to define and compute the similarity between cocktails. 

The most accepted way of representing similarity is computing distance between two entities. Let's ponder on how friends represent one's characteristics in this quote  "Your friends are a reflection of your own personality". We can say that "close" friends share similar characteristics. Family members who are usually closer than your friends even look similar. Taking this analogy into account, **we can define similarity as the distance between entities.** (Smaller the distance, closer the two elements are and the more similar they are) 

TL;DR: We can build a Content Based Recommendation System **based on the similarity between cocktail recipes measured by distance.**

<br>

### Examples of Computing Similarity

Now the complication of the recommendation system lies on how to compute distance between cocktail recipes. 

There are many ways to measure distance and similarity depending on the context. The following are only a few well known methods:

- **Euclidean distance** measures the length of a straight line between two points in the plane.
- **Jaccard distance** measures how dissimilar two sets are by taking the proportion of total number of  non-common elements to the union of two sets. 
- **Word Mover's distance** use word embedding to calculate distance between documents even considering relevant ones without sharing common words. The magic comes from calculating minimum traveling distance between documents by comparing words. This assumes that synonyms will have shorter distance after the embedding. 
- **Cosine similarity** measures the similarity of vectors by computing their inner products. The smaller angle the two vectors share, higher the similarity. 

Among the many methods of measuring similarity which one should we adopt? Should we say that recipes are similar when they share common ingredients? Or should we convert text into numbers with embedding methods and then calculate the distance?

<br>

<br>

## Defining "Similar Cocktails"

When you call something similar, the notion is based on resemblance of specific features. When we say two cocktails are "similar", it refers to the taste, color or ingredients that they share. As our data includes neither taste nor color, we need to make the most out of ingredients. Because flavor depends on the ingredients, we can hope our model to capture the information. For example, Jack and Coke consists of whiskey (Jack Daniel mostly) and coke. Cuba Libre is a mixture of rum and coke. Because the two shares a common ingredient coke, we can expect the two cocktails to be similar in taste. However, the important idea is that the ratio of ingredient should be similar. 0.5oz of coke would not contribute to the flavor as much as 1oz of coke to a 2oz drink. 

We can use **Word Mover's Distance** (WMD) to define the distance between two cocktails as below:

$$Distance_{A,B} =\min_{T\ge0} \sum_{i,j=1}^{n}T_{ij}c(i,j)$$

$$ where:  \\ \bullet \ \ n = \text{a finite size of ingredients in corpus after word2vec embedding}\\\bullet\ T_{ij} = \text{how much of ingredient}\ i\ \text{in Cocktail A travels to ingredient}\ j\ \text{in Cocktail B} \\\bullet\ \sum_{j=1}^{n}T_{ij} = CocktailA_{i^{th}ingredient}, \ \forall i \in \{1,2,...,n\} \\ \bullet\ \sum_{i=1}^{n}T_{ij} = CocktailB_{j^{th}ingredient} , \ \forall j \in  \{1,2,...,n\} \\\bullet\ c(i,j) = cost \ of \ traveling \ from \ one \ ingredient \ to\ another$$



The WMD model can capture combination of pairs of ingredients frequently used together in the cocktail recipes. However, it does not take ratio of ingredients into account. To resolve this problem, we can use **cosine similarity** and calculate distance by averaging the sum of weighted word vectors, like the illustration below:

![cosine]({{site.baseurl}}/assets/img/cosine_cockt.jpg =700x)

The distance/similarity between two cocktails can be computed as the approaches introduced above. We can use ensemble method to use several different models for better performance. Average ranking of the closest cockatils can be retrieved from the results of different distance models, and adopted as our final index for cocktail recommendation.

![recommendation engine diagram]({{site.baseurl}}/assets/img/cocktail_recommend_diagram.jpg =700x)

<br>

## Implementation

### Word Mover's Distance (WMD)



### Cosine Similarity using BERT Embedding





