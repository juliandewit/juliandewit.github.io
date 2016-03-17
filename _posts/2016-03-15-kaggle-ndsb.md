---
layout:     post
title:      "3rd place solution for the second national datascience bowl"
date:       2016-03-15 00:00:00
author:     "Julian de Wit"
---

## Summary
This document describes the 3rd prize solution to the [Second National Data Science Bowl](https://www.kaggle.com/c/second-annual-data-science-bowl) hosted by Kaggle.com. The goal of the challenge was to perform automatic volume measurement of the left ventricle based on MRI images. I will skip the description of the challenge since it has already been described on the [Kaggle competition page](https://www.kaggle.com/c/second-annual-data-science-bowl) and [this blog post](http://irakorshunova.github.io/2016/03/15/heart.html) of the #2. My solution relied heavily on image segmentation with a neural network architecture called [U-nets](http://lmb.informatik.uni-freiburg.de/people/ronneber/u-net/).  

## The plan
After the last [Retinopathy challenge](https://www.kaggle.com/c/diabetic-retinopathy-detection) I really got a taste for solving medical problems. So I was very excited about this. Just to get a feeling for the problem I decided to manually segment a few patients and to compute the volume on paper. With the information from the DICOM files (pixel area, image location etc) one could go directly from pixels to milliliters. Without exaclty knowing how to determine which areas were part of the left ventricle I roughly got an error of ~10ml. I concluded that if I could learn the computer how to segment, it would result in a very good score. Below is the idealised schema for the solution.

Of course things turned out to be a bit harder then expected. First of all, the data was for from perfect. Second it was very hard to find information on how exactly one can determine left ventricle area on MRI images. The sunnybrook database that was provided only contained very simple examples. I decided to extend my system with a data cleaning part and a calibration step. The calibration step uses the provided volumes in the traindata to correct for systematic mistakes made with the automatic image segmenter.








