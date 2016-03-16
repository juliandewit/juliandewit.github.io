---
layout:     post
title:      "My solution for the second national datascience bowl"
date:       2016-03-15 00:00:00
author:     "Julian de Wit"
---

## Summary
This document describes the 3rd prize solution to the [Second National Data Science Bowl](https://www.kaggle.com/c/second-annual-data-science-bowl) hosted by Kaggle.com. The goal of the challenge was to perform automatic volume measurement of the left ventricle based on MRI images. My solution relied heavily on image segmentation with a neural network architecture called [U-nets](http://lmb.informatik.uni-freiburg.de/people/ronneber/u-net/). 

## The challenge
I keep this part to a minimum since it has already been described on the [Kaggle competition page](https://www.kaggle.com/c/second-annual-data-science-bowl) and [this blog post](http://irakorshunova.github.io/2016/03/15/heart.html) of the #2. This team had a completely different approach. To follow my approach it is enough
