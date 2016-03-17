---
layout:     post
title:      "3rd place solution for the second national datascience bowl"
date:       2016-03-15 00:00:00
author:     "Julian de Wit"
---

## Summary
This document describes the 3rd prize solution to the [Second National Data Science Bowl](https://www.kaggle.com/c/second-annual-data-science-bowl) hosted by Kaggle.com. The goal of the challenge was to perform automatic volume measurement of the left ventricle based on MRI images. I will skip the description of the challenge since it has already been described on the [Kaggle competition page](https://www.kaggle.com/c/second-annual-data-science-bowl) and [this blog post](http://irakorshunova.github.io/2016/03/15/heart.html) of the #2. My solution relied heavily on image segmentation with a neural network architecture called [U-nets](http://lmb.informatik.uni-freiburg.de/people/ronneber/u-net/).  

## The plan
After the last [retinopathy challenge](https://www.kaggle.com/c/diabetic-retinopathy-detection) I really got a taste for solving medical problems. So I was very excited about this competition. Just to get a feeling for the problem I decided to manually segment a few patients and to compute the volume on paper. With the information from the DICOM files (pixel area, image location etc) one could go directly from pixels to milliliters. Without exaclty knowing how to determine which areas were part of the left ventricle I roughly got an error of ~10ml. I concluded that if I could learn the computer how to segment, it would result in a very good score. Below is the idealised schema for the solution.

Of course things turned out to be a bit harder then expected. First of all, the data was for from perfect. Second it was very hard to find information on how exactly one can determine left ventricle area on MRI images. The sunnybrook database that was provided only contained very simple examples. I decided to extend my system with a data cleaning part and a calibration step. The calibration step uses the provided volumes in the traindata to correct for systematic mistakes made with the automatic image segmenter. In the next paragraph I will try to discuss every step of the complete solution.

## Preprocessing
The MRI images were provided in the DICOM format. The DICOM files contained an MRI scan and lots of metadata. The preprocessing  consisted of three paths. The first step was to extract the images from the file and make them uniform and easier to see and process.
Second, the datacleaning step needed as much information on how the scanning process took place. Interesting fields were patient-orientation, scan-time, slice-location, pixel-area and a few others. The second step was to find useful metadata fields that could be used for the data-cleaning and calibration step. Examples of interesting features were age, sex, slice-count, slice-distance, patient-orientation, scan-time and a few others.

All images were taken at different scales (reflected in the pixel-area property). My hunch was that this is done to get more uniform pictures of the heart. Small hearts would be zoomed in and bigger hart would be zoomed out. By just looking at the images I still saw a lot of variance and the picture just did not look very uniform. Therefore I decided to scale every image by the pixel-area^2 factor. This was for every image 1 pixel respresented the same area. After scaling the images I noted that the heart was always bounded by a 180x180 area. Usually it's good to leave out irrelevant information for picking up signal by the machine learning algorithm  but it also helps for processing speed. Therefore it was decided to crop all images to 180x180. 







