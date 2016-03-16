---
layout: post
title: Click-through rate prediction using an ensemble of neural networks
date:       2014-09-30 00:00:00
author:     "Julian de Wit"
---

## Summary
This document describes the 4th prize solution to the [Criteo Labs display advertising challenge](https://www.kaggle.com/c/criteo-display-ad-challenge) hosted by Kaggle.com. The solution consisted of two steps. First the data was preprocessed. Rare and unseen test-set categorical values were all encoded as one category.  The remaining features were one-hot encoded or hashed so that we ended up with roughly 200K separate sparse features. The second step was to use neural networks to train a number of different models with variations coming from different network architectures, bagging and preprocessing parameters. The different models were averaged into one final solution.

## the rest will be converted later

[See the original document](https://github.com/juliandewit/kaggle_criteo/blob/master/ModelDocumentation.pdf)

