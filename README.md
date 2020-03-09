
# Project Depth


## INTRODUCTION
Flood is one of the most distructive disasters in the world. More than half of flood disasters occur in 
developing parts of the world, thus exposing the most vulnerable people on planet to its unfortunate effects.
Major causes of flood events include heavy rainfall, high tides and human factors such as deforestation, 
poor waste management, irregular urbanization, among others. With the aggravating effects of climate change 
set to rise in the coming years, 
flooding risks might also become significant in developed countries.

Since the risks of flood are majorly concentrated in low-resource countries, 
solutions and ways of managing this problem must also be of very low cost. 
One very revevant intervention is the use of available data sources to track human falalities and direct intervention and rescue operations.
In the past, remotely sensed data from satellites have been used to retrieve  flood-water level from the disaster areas. 
This technique, though inexpensive, doesn't provide near real-time monitoring capabilities. This is because freely available remotely sensed
 data which are usable for this purpose are available at relatively low temporal resolution. 
Other techniques that have been used also include stream guage and field data collection. 
These techniques are both expensive.

A viable alternative is the use of real-time mobile phone camera images or images from 
social media platforms readily annotated with user-shared texts.
These data can be available for free and in real-time, 
however they require heavy cleaning, annotation and pre-processing in other to make them meaningful.
Other researchers already addressed this problem. 
Following from what they have done, we intend to the capability of the solution by fitting a similar model, 
with custom collected and annotated data, 
and it's Deployment.

In this document, we will discuss the background and motivation of our study: 
challenges that arise from creating a such segmentation model for real-time flood location images and deploying it on the edge.
We describe our processing and modeling pipeline and the final edge application deployment.


## METHODOLOGY

The Project is Divided into two phases :
•	Development of Deep Learning Framework
•	Deployment of Project at the Edge.

For the model development phase the steps to be followed are briefly Described below with reference to the paper .
However the work has to be done from Scratch since no Dataset are formal Code for the framework has been released.



1.	Data Collection and Annotation -There are two Dataset used for this project.
•	In order to feed the annotated datasets to the network, we use MS COCO object detection annotation format (COCO, n.d.). We extract the information for various fields from the annotated image and generate mask for each object instance using MS COCO API.The annotations are stored using JSON .

•	The second dataset is a custom build flood Dataset which contains images of flood instances from around the globe collected and compiled by utilizing the publicly available datasets and images from the web.The dataset will be annotated pixelwise, using an online annotation tool called Supervisely.

2.	Annotation  Strategy - In order to train our neural network we need the images in our training dataset to be labeled for all the four quantities we want to predict. As the goal of this study is to quantify flood-water level based on objects partially submerged in water, the first step for defining the annotation strategy is to decide which objects we should consider for the classification task. The criteria for selecting the objects for this task are: easy availability, known dimensions, and low intra-class height variance. By easy availability, we mean objects which are common in the real world, so that it is easy to gather a large number of pictures containing the object, both for training and prediction. Known dimensions refer to the fact that height, length and width of an object are approximately known. Finally low variation in height means that several instances of the same object in the real world have approximately the same height. For instance, bicycles are objects that are extremely common in urban environments, we roughly know their size, and their height is roughly constant across different models. Based on the criteria we decided to consider these five classes of objects: Person, Car, Bus, Bicycle, and House. In addition to the five classes mentioned above, we also consider the flood class, which represents flood-water present in the image.
The one label we are missing is the one for the water level prediction. To obtain this label we need a course of action to quantify the flood-water height. As humans cannot just by looking at an image deduce the centimetres of flood-water, we decided to pursue a strategy that tries to estimate how much of an object body is submerged in water in terms of some coarsely defined levels. Since the main concern in case of floods is to prevent human fatalities, it makes sense to consider the human size as main building block for this prediction. We consider 11 flood levels, levels go from 0, which means no water, to 10, which represents a human body of average height completely submerged in water. Moreover, since in order to create the training dataset, we need to annotate manually the images with water level information, it is important to select levels that facilitate this operation. The height of the different levels is then inspired by drawing artists who use head height as the building block for the human figure. To map level classes to actual flood height, we consider an average height human body and derive the water height in cm, see Table 1. We can now extend the annotation strategy to the other four different classes of objects by considering their average height. After getting an approximate estimate of the height of these objects in the real world, we compare them with the average human height, on which the 11 flood levels are defined, and extend the flood level definition to these other objects. In below figure we show how, the flood levels defined for a human body, translate for an average size Bicycle.

![Respective Comparison](https://drive.google.com/file/d/1HU-bel40i8pRvEg98_HbL9UwY_MVMhub/view?usp=sharing)





## REFERENCES
https://www.analyticsvidhya.com/blog/2019/07/computer-vision-implementing-mask-r-cnn-image-segmentation/

https://www.analyticsvidhya.com/blog/2018/07/building-mask-r-cnn-model-detecting-damage-cars-python/

https://towardsdatascience.com/region-of-interest-pooling-f7c637f409af
