# **Behavioral Cloning**

## Writeup

---

**Behavioral Cloning Project**

The goals / steps of this project are the following:
* Use the simulator to collect data of good driving behavior
* Build, a convolution neural network in Keras that predicts steering angles from images
* Train and validate the model with a training and validation set
* Test that the model successfully drives around track one without leaving the road
* Summarize the results with a written report


[//]: # (Image References)

[nvidia]: ./examples/nvidia-cnn-architecture.png "Model Visualization, from Nvidia blog post"
[architecture]: ./examples/architecture.png "Model Visualization, custom-drawn with whiteboard"
[center]: ./examples/center.jpg "Center driving"
[image3]: ./examples/recovery1.jpg "Recovery Image"
[image4]: ./examples/recovery2.jpg "Recovery Image"
[image5]: ./examples/recovery3.jpg "Recovery Image"

## Summary
**Explain a recent project you've worked on. Why did you choose this project? What difficulties did you run into this project that you did not expect, and how did you solve them?**

Recently, I used Behavioral Cloning to drive a car around a racetrack. I chose this project since it was part of the Udacity Nanodegree and sounded very interesting to teach a car to drive based on my own driving data. 

I used Keras to make a CNN with five conv layers and three fully-connected (FC) layers. I used dropout layers between the FC layers to reduce overfitting, since dropout sets some values to 0, thus reducing the prediction's dependence on specific connections.

![Model Architecture][architecture]

Preprocessing:
- Crop
- Normalize image values (RGB)
- DON'T normalize steering angles — visualize to make sure distribution is roughly Gaussian — outputs expected to be heavier left side (left turning) when testing. Pre-scaled to [-1, +1].

**Feature engineering**: Input image + steering angle | Left image (adjust steering angle right by a constant) | Right image similarly

**Cleaning data**: The data didn't have smooth steering angles — it took a lot of training practice to get this right.

- Some unexpected difficulties was in collecting training data. Each sample consisted of an input image from center camera and a steering angle. When the camera shows the car is in the center of the road, don't steer; but if it's, say right of center or approaching a leftward curve, steer to the left. Data samples are collected using a Unity simulator. However, my driving was initially not good as I would overcorrect or react unpredictably instead of providing smooth steering angles, so much of the my training data did not have ideal or even good angles.
- I visualized the steering angle distribution by plotting to see how abrupt my turning was. I removed samples with steering that was particularly abrupt.
- Another strategy I used was integrating mouse-based steering instead of keyboard-based steering (using simulator code mentioned on the forum). This reduced the abruptness from keyboard presses, as mouse movement is more continuous.
- Sometimes the car would miss turns or wobble a lot on straight roads. I needed to collect a lot more data than expected (drive around laps) and even compile new datasets to reflect changes in my driving approach. I made multiple datasets of driving around multiple laps. I also collected some datasets mostly on turns so the car can navigate curves correctly. I alternately used my best turns dataset with my best multiple-lap dataset to train my model, using lower learning rates to fine-tune it.
- Eventually, my car was able to complete multiple laps around the track smoothly and autonomously.

**Follow-up: How else would you generate good data, besides just practicing a lot?**

- Aforementioned is plotting steering angles as time-series data and comparing with sample training images. Remove particularly abrupt (bad) data and be careful retraining in those segments.
- One idea is I could have different people generate data and see how the model performs on those — they might use smooth steering angles from the get-go, and have steadier hands than I do. I could also combine data if they have different strengths. For example, if someone drives very well around corners, I could collect a lot of that data and combine it with the non-corners data I have.
- Since I was at home, I asked my dad to do some driving but he wasn't better than me as far as the simulator was concerned (need to be _very_ subtle when steering to prevent outliers). I'd probably rely on this more in industry, where I have many colleagues nearby to use my simulator and it would be a fun break for them. And we'd also generate more ideas by experiencing this driving and talking together.
- Untested idea: use Huber loss to mitigate abrupt driving samples



## Rubric Points
### Here I will consider the [rubric points](https://review.udacity.com/#!/rubrics/432/view) individually and describe how I addressed each point in my implementation.

---
### Files Submitted & Code Quality

#### 1. Submission includes all required files and can be used to run the simulator in autonomous mode

My project includes the following files:
* `model.py` containing the script to create and train the model
* `drive.py` for driving the car in autonomous mode
* `model.h5` containing a trained convolution neural network
* `writeup.md` summarizing the results
* `run1.mp4`, a recording of my vehicle driving autonomously around the track for at least one full lap

#### 2. Submission includes functional code
Using the Udacity provided simulator and my drive.py file, the car can be driven autonomously around the track by executing
```sh
python drive.py model.h5
```

#### 3. Submission code is usable and readable

The `model.py` file contains the code for training and saving the convolution neural network. The file shows the pipeline I used for training and validating the model, and it contains comments to explain how the code works.

### Model Architecture and Training Strategy

#### 1. An appropriate model architecture has been employed

My model consists of a convolutional neural network based on the Udacity lectures, Q&A, and mentioned NVIDIA CNN (["End-to-End Deep Learning for Self-Driving Cars"](https://developer.nvidia.com/blog/deep-learning-self-driving-cars/)). It consists of five convolutional layers and three fully-connected layers.

![alt text][nvidia]

The data is first normalized in the model using a Keras Lambda layer. Then a Keras Cropping2D layer crops the images to remove the noisy scenery (top of image) and car hood (bottom of image), only keeping the road terrain.

The model uses a RELU layer after each Keras Conv2D layer to introduce nonlinearity.

#### 2. Attempts to reduce overfitting in the model

The model originally contained several dropout layers in order to reduce overfitting, one after each convolutional layer (drop prob of 0.1-0.3) and after each fully-connected layer (drop prob of 0.5). That was the idea per https://stackoverflow.com/a/47959567/6293259 and https://stackoverflow.com/a/47959567/6293259, but the final model is simplified from this, as the model was not converging quickly enough from the training data on crucial turns.

The final model architecture submitted here uses two dropout layers after two hidden fully-connected layers.

The model was trained and validated on disjunct data sets to ensure that the model was not overfitting. These were split using `sklearn.model_selection.train_test_split`. There is no specific test set; the model was tested by running it through the simulator and ensuring that the vehicle could stay on the track. Proof is provided by the video.

#### 3. Model parameter tuning

The model used an Adam optimizer, so the learning rate was not tuned manually, for the most part. Later training used fine-tuning to incorporate new turn data without upsetting model performance on straight sections (e.g. feed Adam with 1e-4 learning rate).

Due to GPU Out of Memory errors at points, I changed batch size to 16.

I increased my steer_correction factor for left/right images, since my model was going straight too often, so I thought a more extreme steering adjustment might help (also I originally tested at higher speeds than my training data, and the car was not steering enough at certain turns)

#### 4. Appropriate training data

Training data was chosen to keep the vehicle driving on the road. I started with some data of center lane driving, then recorded much data recovering from the left and right sides of the road, especially in the areas the autonomous driving model failed in.

For details about how I created the training data, see the next section.

### Model Architecture and Training Documentation

#### 1. Solution Design Approach

The overall strategy for deriving a model architecture was to start with the suggested architectures and improve upon them.

My first step was to use a convolutional neural network model similar to the NVIDIA model mentioned previously. I thought this model might be appropriate because it was used and regarded as effective for their similar subject of self-driving car behavioral cloning.

In order to gauge how well the model was working, I split my image and steering angle data into a training and validation set. I found that my first model had a low mean squared error on the training set but a high mean squared error on the validation set. This gap between loss values implied that the model was overfitting. To combat this overfitting, I modified the model to add dropout layers after the conv and fully-connected layers. Then I trained this new model with my training data.

The final step was to run the simulator to see how well the car was driving around track one. There were a few spots where the vehicle fell off the track, including the brown turn after the bridge. (Getting past this turn was by far the most time-consuming step...) To improve the driving behavior in these cases, I created more training data at those specific turns. And then more. And then more. Lots of models, lots of retraining, an endless loop of madness! I sometimes fell into the trance of the Strange Loop of the track. At 2AM the Udacity workspace froze and my data in `/opt` was gone forever. Lots of data, retraining, ...  Somehow, the car would not collide into rectangular barriers. Eventually, it would actually turn _while in the center of the road_. Amazing. I've finally learned how to drive better than my desired CNN! And so I became one with the machine. Is it a dream? Am I inside this world now? Lots of data, retraining, ...  I digress.

At the end of the process, the vehicle is able to drive autonomously around the track without leaving the road. _clap_ _clap_

#### 2. Final Model Architecture

The final model architecture consisted of a CNN with five convolutional layers and three fully-connected layers, utilizing ReLU activation layers to introduce nonlinearity. It has been described earlier.

Here is a visualization of the architecture. Details of the architecture could be seen here (some parts were modified in my model, most notably the presence of dropout layers; refer to code in `model.py`).

![alt text][architecture]

#### 3. Creation of the Training Set & Training Process

My first lap, I had difficulty staying on the road and went over some lane lines, and later laps had slightly better training data. Afterwards, to deal with vehicle oscillations, I focused much more on the recovery from left + right sides of the road. Next, noticing the model performed glaringly poorly in some turns, I focused in collecting data for those turns, using a separate data directory to keep epoch-training times lower.

To capture good driving behavior, I first recorded two laps on track one using center lane driving. Here is an example image of center lane driving:

![alt text][center]

I then recorded the vehicle recovering from the left side and right sides of the road back to center so that the vehicle would learn to steer back towards center when it is off to the sides. These images show what a recovery looks like starting from being oriented towards the edge of the road:

![alt text][image3]
![alt text][image4]
![alt text][image5]

To augment the data sat, I also horizontally flipped images and angles. Since I went counter-clockwise around the track for my training data, this flipping would replicate going clockwise around the track.

After the collection process, I had 25k+ data points. Data augmenting via horizontal-flipping doubled this amount. I then preprocessed this data by cropping out the top and bottom portion of each image, leaving only the salient portion of the road.

I finally randomly shuffled the data set and placed 10% of the data into a validation set, formed using sklearn.model_selection.train_test_split.

I used this training data for training the model. The validation set helped determine if the model was over or under fitting. I used an Adam optimizer so that manually tuning the learning rate wasn't necessary.

__Appendix on Training difficulties (ranting)__

* Multiple data sets with varying amounts of center-road driving and responding to particular turns (car would often crash into the diamond-shaped corners of the bridge or the brown road after -- it seemed very attracted to corners). I took a _lot_ of data to correct turns at these places.
* I also tried cropping the images to various amounts and even resizing (to speed up training and avoid GPU Out of Memory errors -- my model performed poorly with these resizes and I found "Refresh Workspace" resolved GPU OOM, especially if I was testing the model concurrently with performing training, e.g. with that model file loaded).
* I was saving my data in /opt/carnd_p3/turns_data and one time the machine froze and everything in /opt/carnd_p3 was reset, losing all my data
* I created multiple models of varying architectures and levels of training of each of the dataset, to compare driving behavior. Finally, I arrived at one which was smooth and did not attract itself to edges. Data collecting took ~10 hours in total, and this was entirely aimed to get the car to complete a loop around the first track.
* The car would frequently swerve in straight sections (especially the early one) at my desired 30mph. So I recorded the video at an agonizingly 10mph so the vehicle could respond fast enough and avert the edge-attracting disasters that has long plagued these 15~ GPU workspace hours.
* Related to all of the above: did a lot of dataset cleaning by visualizing the data (plotting steering and image). Having to delete data at a lot of turns (due to abrupt steering) made me collect a lot more data much more carefully. This analysis took about as long as training (smoothening peaks of data, or just removing swaths in the interest of collecting more data). This was done very ad-hoc, as I noticed training on more turns data actually increased oscillating -- could have done this way more structured and evaluated my training data after each (including the first) collection.
* At one point, I gave up collecting my own data (I had difficulty keeping to center of road and steering well myself). I got my dad to gather some data. That helped quite a lot in smoothing turn angles (staying to center, turning early). (after purging the abrupt segments of course)
