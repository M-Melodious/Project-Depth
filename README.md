
# FLOOD LEVEL ESTIMATION USING MOBILE PHONE GENERATED IMAGES


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
and deploying it on the edge using Intel® OpenVINO™ Toolkit. 

In this document, we will discuss the background and motivation of our study: 
challenges that arise from creating a such segmentation model for real-time flood location images and deploying it on the edge.
We describe our processing and modeling pipeline and the final edge application deployment using Intel® OpenVINO™ Toolkit. 


## DATA COLLECTION AND ANNOTATION



## METHODOLOGY
We use Mask R-CNN as the core of our methodology. This decision was taken for the following reasons:
* It is a state-of-the-art solution for instance segmentation.
* It has shown good results in a [similar study](https://www.isprs-ann-photogramm-remote-sens-spatial-inf-sci.net/IV-2-W5/5/2019/isprs-annals-IV-2-W5-5-2019.pdf). 

The standard Mask R-CNN architecture comprises the backbone, region proposal network , proposal classification and mask generation blocks.







## REFERENCES
https://www.analyticsvidhya.com/blog/2019/07/computer-vision-implementing-mask-r-cnn-image-segmentation/

https://www.analyticsvidhya.com/blog/2018/07/building-mask-r-cnn-model-detecting-damage-cars-python/

https://towardsdatascience.com/region-of-interest-pooling-f7c637f409af