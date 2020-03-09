
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

For the model development phase the steps to be followed are briefly Described below with reference to the paper while the deployment on edge is something that has never been done before.

However the work has to be done from Scratch since no Dataset are formal Code for the framework has been released.

### PHASE 1 : MODEL DEVELOPMENT.


1.Data Collection and Annotation -There are two Dataset used for this project.

- In order to feed the annotated datasets to the network, we use MS COCO object detection 	annotation format (COCO, n.d.). We extract the information for various fields from the 		annotated image and generate mask for each object instance using MS COCO API.The annotations 	are stored using JSON .

- The second dataset is a custom build flood Dataset which contains images of flood instances from around the globe collected and compiled by utilizing the publicly available datasets and images from the web.The dataset will be annotated pixelwise, using an online annotation tool called Supervisely.

2.Annotation  Strategy - In order to train our neural network we need the images in our training dataset to be labeled for all the four quantities we want to predict. As the goal of this study is to quantify flood-water level based on objects partially submerged in water, the first step for defining the annotation strategy is to decide which objects we should consider for the classification task. The criteria for selecting the objects for this task are: easy availability, known dimensions, and low intra-class height variance. By easy availability, we mean objects which are common in the real world, so that it is easy to gather a large number of pictures containing the object, both for training and prediction. Known dimensions refer to the fact that height, length and width of an object are approximately known. Finally low variation in height means that several instances of the same object in the real world have approximately the same height. For instance, bicycles are objects that are extremely common in urban environments, we roughly know their size, and their height is roughly constant across different models. Based on the criteria we decided to consider these five classes of objects: Person, Car, Bus, Bicycle, and House. In addition to the five classes mentioned above, we also consider the flood class, which represents flood-water present in the image.
The one label we are missing is the one for the water level prediction. To obtain this label we need a course of action to quantify the flood-water height. As humans cannot just by looking at an image deduce the centimetres of flood-water, we decided to pursue a strategy that tries to estimate how much of an object body is submerged in water in terms of some coarsely defined levels. Since the main concern in case of floods is to prevent human fatalities, it makes sense to consider the human size as main building block for this prediction. We consider 11 flood levels, levels go from 0, which means no water, to 10, which represents a human body of average height completely submerged in water. Moreover, since in order to create the training dataset, we need to annotate manually the images with water level information, it is important to select levels that facilitate this operation. The height of the different levels is then inspired by drawing artists who use head height as the building block for the human figure. To map level classes to actual flood height, we consider an average height human body and derive the water height in cm, see Table 1. We can now extend the annotation strategy to the other four different classes of objects by considering their average height. After getting an approximate estimate of the height of these objects in the real world, we compare them with the average human height, on which the 11 flood levels are defined, and extend the flood level definition to these other objects. In below figure we show how, the flood levels defined for a human body, translate for an average size Bicycle.

![Respective Comparison](https://github.com/DemocraticAI/Project-Depth/blob/master/images/man.PNG)

![Estimate Conversion Table](https://github.com/DemocraticAI/Project-Depth/blob/master/images/level.PNG)


3.	Neural Network Architecture : In this section, we describe the deep learning approach used for flood-water level estimation. We use Mask R-CNN as base architecture which is a state-of-the-art solution for instance segmentation. Below Figure illustrates the overall architecture of the method. The backbone of the architecture works as the main feature extractor. We can use any standard convolutional neural network. The idea is to pass an image through various layers which extract different features from the image. The lower layers detect low-level features like blobs, edges. As we move to higher layers, they start detecting full objects like cars, people, buses. The input image gets converted to feature maps in this module for an easier handling in the other modules. The above described backbone can be improved upon using Feature Pyramid Network(FPN) which was introduced by the same authors of Mask R-CNN .FPN represents objects at multiple scales better by passing the high level features from first pyramid down to lower layers of second pyramid. This allows features to have access to both lower and higher level features. We use ResNet101 and FPN as our backbone.

![Deep Learning Neural Network Architecture](https://github.com/DemocraticAI/Project-Depth/blob/master/images/architecture.PNG)


4.	Evaluation Strategy : As our network generates four different predictions (class, bounding box, level, and mask prediction) but ultimately we only care about the level prediction, we need to separate the performance of the object detector and level classifier. There can be two scenarios: False Positive(FP) and False Negative(FN) detections. If an object instance is not detected, there will also be no level predictions. In simple terms, the ground-truth file stores for every image, the bounding box, class label, level label, and mask, for all object instances present in the image. During prediction, if one or more of these object instance(s) is not detected there will also be no level label prediction. This scenario corresponds to the False Negative(FN) case. Similarly, False Positive cases are also a possibility, in this case an object detector wrongly detects background (image area where no object lies) as an object and predicts a class label for background. If a class of an object is wrongly predicted, it is also considered a FP case.
We describe a method to compute a global image water level from the individual object predictions. The reason for doing this is that for our purpose, we ultimately do not need a level value for each object instance, as it is too detailed, but an overall score for the entire image might be sufficient. The question that obviously arises is why not just train the network to give a single value for every image. At first we investigated this naive approach which, however, performed poorly. It is in fact very complicated to predict directly a water level for the entire image.

For the calculation of the global water level we compute the trimmed mean of the predicted levels of the different object instances.
Once the model is trained and evaluated it needs to be Deployed. 

### Phase 2 : Model Deployment.


•	The main motive of using the Edge Technology behind the deployment of this Project is it’s use case, Disaster Prone Areas almost get disconnected from the outer world .

•	Being able to process the image in runtime within fraction of second is another advantage here.

•	Optimization software, especially made for specific hardware, can help achieve great efficiency with edge AI models

•	Disaster can occur anywhere be it Rich or Poor State hence being able to deploy an affordable solution becomes a mandate in such scenarios.

We wanted to deploy our model specifically on mobile or hand sets to make it easy and out reachable to the maximum of the people and locations,however after hobnobbing for a while in the Intel’s Toolkit of OpenVINO we unfortunately could not figure out the methodology of deployment of model on mobile phones as so far it does not allows it.

So to make it happen we choose TensorFlow Lite for our purpose ,
The Phases Of Deployment Can be divided into the following steps:

Below Figure shows a Guided Pipeline of the Deployment Process.

![Deployment Architecture](https://github.com/DemocraticAI/Project-Depth/blob/master/images/tflite%20arch.PNG)

Pre-processing :

1)	Model Conversion

Since we have created our own model here we need to pass our trained model for the process of Conversion. TensorFlow Lite is designed to execute models efficiently on mobile and other embedded devices with limited compute and memory resources. Some of this efficiency comes from the use of a special format for storing models. TensorFlow models must be converted into this format before they can be used by TensorFlow Lite.
Converting models reduces their file size and introduces optimizations that do not affect accuracy. The TensorFlow Lite converter provides options that allow you to further reduce file size and increase speed of execution, with some trade-offs.
TensorFlow Lite converter
The TensorFlow Lite converter is a tool available as a Python API that converts trained TensorFlow models into the TensorFlow Lite format. It can also introduce optimizations, which are covered in section 3, Optimize your model.




2)	Run inference with the model

Inference is the process of running data through a model to obtain predictions. It requires a model, an interpreter, and input data.
TensorFlow Lite interpreter
The TensorFlow Lite interpreter is a library that takes a model file, executes the operations it defines on input data, and provides access to the output.
The interpreter works across multiple platforms and provides a simple API for running TensorFlow Lite models from Java, Swift, Objective-C, C++, and Python.

Android and iOS:The TensorFlow Lite interpreter is easy to use from both major mobile platforms.

3)	Optimize your model

TensorFlow Lite provides tools to optimize the size and performance of your models, often with minimal impact on accuracy. Optimized models may require slightly more complex training, conversion, or integration. The goal of model optimization is to reach the ideal balance of performance, model size, and accuracy on a given device.
We choose Quantization as a process of optimization for our purpose of Optimzation.
- Quantization reduces the precision of values and operations within a model, quantization can reduce both the size of model and the time required for inference. For many models, there is only a minimal loss of accuracy.

TensorFlow Lite supports reducing precision of values from full floating point to half-precision floats (float16) or 8-bit integers. There are trade-offs in model size and accuracy for each choice, and some operations have optimized implementations for these reduced precision types.

Deployment in Hardware:
For now we are showing the deployment process for Android only with reference to Android Studio,likewise the deployment can be done in iOS as well.

1)	Accessing the Model : Downloading, extracting, and placing the model in the assets folder is managed automatically by download.gradle.
2)	Build and Run:

•	Step 1: Clone the TensorFlow examples source code.

Clone the TensorFlow examples GitHub repository to your computer to get the demo application.
`git clone https://github.com/model`

Open the TensorFlow source code in Android Studio. To do this, open Android Studio and select Open an existing project, setting the folder to `examples/lite/examples/image_classification/android`

![ConfigPic1](https://github.com/DemocraticAI/Project-Depth/blob/master/images/1and.PNG)



• Step 2:	Build the Android Studio Project.

Select `Build -> Make Project` and check that the project builds successfully. You will need Android SDK configured in the settings. You'll need at least SDK version 23. The `build.gradle` file will prompt you to download any missing libraries.
The file `download.gradle` directs gradle to download the two models used in the example, placing them into assets.

![ConfigPic2](https://github.com/DemocraticAI/Project-Depth/blob/master/images/2and.PNG)


• Step 3:

Install and run the app:
Connect the Android device to the computer and be sure to approve any ADB permission prompts that appear on your phone. Select `Run -> Run app`. Select the deployment target in the connected devices to the device on which the app will be installed. This will install the app on the device.

![ConfigPic3](https://github.com/DemocraticAI/Project-Depth/blob/master/images/3and.PNG)

![ConfigPic4](https://github.com/DemocraticAI/Project-Depth/blob/master/images/and.PNG)

To test the app, open the app called `TFL Classify` on your device. When you run the app the first time, the app will request permission to access the camera. Re-installing the app may require you to uninstall the previous installations.

With Intel’s OpenVINO toolkit everystep remains same and can be followed for same project for other hardware devices, the architecture of which has been given below however the model deployment at Android was not possible.

The process of Intel OpenVINO toolkit has been showed below.

![OpenVINO toolkit process for Edge](https://github.com/DemocraticAI/Project-Depth/blob/master/images/Capture.PNG)
![Inference Engine](https://github.com/DemocraticAI/Project-Depth/blob/master/images/INFERENCE%20ENGINE.PNG)

# RESULT

The Result from reference to the paper can be expected as :

Input Image 

![raw_image1](https://github.com/DemocraticAI/Project-Depth/blob/master/Input/raw1.PNG)

Output Obtained

![output](https://github.com/DemocraticAI/Project-Depth/blob/master/Output/done1.PNG)

## REFERENCES

https://www.tensorflow.org/lite

https://www.analyticsvidhya.com/blog/2019/07/computer-vision-implementing-mask-r-cnn-image-segmentation/

https://www.analyticsvidhya.com/blog/2018/07/building-mask-r-cnn-model-detecting-damage-cars-python/

https://towardsdatascience.com/region-of-interest-pooling-f7c637f409af

https://www.tensorflow.org/lite/guide/android

