---
layout: post
title:  "How to prepare data for analysis"
date:   2017-05-13 09:00:00
author: Hao WU
categories: machine-learning
---

### Introduction

Data is messy, most case, in a unclear and disorganized state, I assume the most important thing before we dive deep into data application is to reconstruct the data we have.

Since the domain "Machine Learning" is waking up in the lightening so that we are exausted to chase its forwarding steps, while on the other hand, less people notice that in fact the thing also matters to the analysis is the materials that Machine Learning algorithms or Models consumes, which you should make straight forward and "understandable".

With this purpose, I'm here to try to give some hints and tricks for data preparation.


### Actions and tips

* Construct OLAP system (Data Warehouse / Data Lake, [Star Model or Galaxy Schema](http://www.vertabelo.com/blog/technical-articles/data-warehouse-modeling-the-star-schema)) from OLTP

> In most cases, the transactional database is the one you or your team working on, and the schema will be extremely difficult for analysis (OTLP is designed for easier and faster writing data rather than reading or even analyzing). So we need to transform such customer business oriented data model to analysis business oriented model of course for analysis purpose. So that we can analyze the data in the normal BI approach at least (Basically roll-up and drill down in the different dimensions and perspectives). In addition, through the data model transformation process, you'll get a deeper impression about the data you have, which will definitely help.

* Feature and Label <-> Column

> As we all known supervised learning or unsupervised learning or else, they cannot directly accept the data with schema. Specifically, machine only accepts 1 big table, which contains all features should be analyzed and learnt, and perhaps also the label to those features if it's supervised learning. So after the OLAP system we constructed, we need a further data observation, do a further roll-ups or drill down for example into the result set, then we can go on.

* Numerical feature

> So currently after the previous 2 operations, we have a big table to go deeper. In the normal daily data, we can see different kind of features, the first type is the numerical ones, for example: housing price for a given selling house, the gray degree of a given pixel, the customer age, etc..

* Non-ordered categorical feature

> Also we used to see lots of non-ordered categorical feature, like color (red, gray, blue), weather (cloudy, sunny, rainy), etc..

* Ordered categorical feature

> And lastly we we ordered categorical features, like ticket class (A, B, C), age stage (child, adult, elder), etc..

* Transform the categorical feature

> Since machine only accepts features in numerical format, which means, machine cannot understand categorical features, so we need to transform categorical ones to numerical. Firstly you can try direct manner, which just to mark categorical feature from (A, B, C) to (1, 2, 3), for both non-ordered and ordered categorical features. It makes more sense in ordered categorical feature, because which brings direct linear impact on the result; while the non-ordered ones brings less value in this way, but it worth trying to see the feasibility. And if the result is not acceptable, we also can try the second way, to convert categorical ones to sparse vectors. Let's say we can split 1 color feature to 3 features: isRed, isGray, isBlue. This way is more complex, but makes sense because each category is separated to independent features.

* Observe and further process data

> Right now we have one big table with all numerical features, but in fact, we often are confused by some features for example "age", which may has no linear relationship with result. Let's imagine age 20 person and age 40 person, they definitely don't have twice difference to each other, so maybe we should transform age feature to age range feature, which may give more clear sense. And besides of transforming 1 feature to another, we also can combine several features to one or several different features.

* Deal with empty data

> It's very often that some records or training samples have empty features, I think if this feature contains very few of empty data, let's say 0.01%, we can just drop them; Otherwise, we can put the average value on the empty cases. 

After all actions above taken, I think we can say firstly we got a machine consumable dataset, and also we have a good knowledge about the data which will help us for the further model tuning. 

In the end, thanks for spending your time on this artical, I hope this artical can do a little help to you, and wish all of you have a nice journey on the machine learning domain!