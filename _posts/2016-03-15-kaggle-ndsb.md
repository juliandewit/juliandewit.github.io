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
The MRI images were provided in the DICOM format. The DICOM files contained an MRI scan and lots of metadata. The preprocessing  consisted of two paths. The first path was to extract the images from the file and make them uniform and easier to see and process.
All images were taken at different scales (reflected in the pixel-area property). My hunch was that this is done to get more uniform pictures of the heart. Small hearts would be zoomed in and bigger hart would be zoomed out. By just looking at the images I still saw a lot of variance and the picture just did not look very uniform. Therefore I decided to scale every image by the pixel-area^2 factor. The result was that for every image 1 pixel respresented the same area. After scaling the images I noted that the heart was always bounded by a 180x180 area. Usually it's good to leave out irrelevant information for picking up signal by the machine learning algorithm  but it also helps for processing speed. Therefore it was decided to crop all images to 180x180. Last but not least, for my own viewing pleasure and to hopefully help the machine learning algorithm I applied contrast stretching to the images using CLAHE (tilesize 1x1). 

The second preprocessing path was to find useful metadata fields that could be used for the data-cleaning and calibration step. Examples of interesting features were age, sex, slice-count, slice-distance, image-orientation, scan-time, image-size and a few others. The features were stored in a separate csv file for later use.

## Hand labeling
The traindata of the competition only contained the final determined volumes of the left ventricle. There was no information on how this volume was obtained. there was the possibility of using the SunnybrookLINK dataset but the cases in that set were much more simple than the cases I encountered in the DICOM files. Of course I was thinking of using a convolutional neural network (CNN) for the image segmentation. CNN's are great but they not a lot of traindata. Luckily I already had a overlay-drawing tool that I once built for an earlier project. I decided to label 100 patients with this tool. For every patient I took frame 1 (usually diastole) and frame 12 (usually systole). 

Since every patient had around 10 slices that meant that I had to label around 2000 images. Once I got the hang of it using my tool I was able to label around 1 patient per minute. During the labeling I encountered many confusing cases that I still don't know how to label. I scanned youtube and many papers but there were no definate answers on the internet. In the end I decided to at least be consistent. If I would be consistent then in a later stage I could brush out systematic errors during the calibration phase. Below are a number of situations that were encountered. 

After I trained a segmented on the 100 patients and made a first submission (#3 at that time while the competition was already 2 months underway my confidence of my approach grew. That motivated me to improve my segmenter and label more images. Most people call this boring work but I actually enjoyed the labeling. Even if you are out of ideas you still have the feeling that you are making progress. Also many high-payed overworked cardiac experts must devote a large portion of their time to this work. I figured that if I did my work right my effort would be neglectable to the effort that could be spared with the final solution. I did However notice more and more diminishing returns for every new batch I labeled.

Like everyone I started to notice the outlier patients that were the biggest barrier to get a higher score. At first I wanted to apply [active learning](https://en.wikipedia.org/wiki/Active_learning_(machine_learning)) to boost hard examples by taking images from other frames than 1 and 12. However, with all the discussion on the forums about wat was and was not allowed I was afraid that people might think that I was actively targetting the test-test with the extra examples so I dropped this idea. However, I think an improvement is possible by boosting hard examples.

## Image segmentation with the U-net architecture (built in MxNet)
When search the literature 2 months ago on image segmentation with CNN's there is no clear "winner" architecture. There were the reasonably successful [sliding widow approaches](http://people.idsia.ch/~ciresan/data/ISBI2012.pdf). However, from first hand experience I knew they were a bit cumbersome to use and far from perfect. The benchmark example provided by Mike Kim used a [fully convolutional neural net](http://www.cs.berkeley.edu/~jonlong/long_shelhamer_fcn.pdf). These are easier to use but they tend to give a rather coarse resolution when you go deep. There are some papers about using [conditional random fields or recurrent nets](http://www.robots.ox.ac.uk/~szheng/papers/CRFasRNN.pdf) as a post processing to improve the detail. However, both approached did not really appeal to me due to the added complexity. Also in this example there were only two classes. [Here](https://github.com/kjw0612/awesome-deep-vision#semantic-segmentation) us an exaustive list of various approaches.

It was a lucky accident that I stubled upon the paper on the [u-net architecture](http://lmb.informatik.uni-freiburg.de/people/ronneber/u-net/). This architectured allowed for more detail in the segmentation by using shortcut connections from the i'th layer to the n-i'th layer. They presented a number of impressive results and as far as I could see the person that came up with the architecture now works for Google Deepmind. To me that proved that appearantly he was on to something.  Below is a schematic overview.

The paper was very clear and readable and I had a prototype running in [mxnet](https://github.com/dmlc/mxnet) without much effort. As someone that deploys deep learning systems to various targets (embedded, windows, linux, cloud etc) in various programming languages (python, c++, c#) I really recommend this library. 

After getting an example running I started to experiment with different parameters and settings. Below a few key observations and improvements are enumerated. 

* Segmentation nets are less numerically stable than normal classification/regression net. I relied heavily on batch normalization.
* Smaller batch sizes yielded better scores and were more numerically stable. I settled for batchsize = 2 in the end.
* The shortcut connection give a significant improvement during training. 
* I used mean squared error as a loss function but that was not optimal. Esp. since different imagesizes gave uncomparable results
* Heavy dropout helped for generalization but it was hard to determine where to apply it. I choose upstream after the shortcut merges.
* Adding more layers unexpectedly quickly led to diminishing returns
* Adding more filters per layer quicky showed no improvement















