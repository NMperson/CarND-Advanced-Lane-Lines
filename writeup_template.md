##Writeup Template
###You can use this file as a template for your writeup if you want to submit it as a markdown file, but feel free to use some other method and submit a pdf if you prefer.

---

**Vehicle Detection Project**

The goals / steps of this project are the following:

* Perform a Histogram of Oriented Gradients (HOG) feature extraction on a labeled training set of images and train a classifier Linear SVM classifier
* Optionally, you can also apply a color transform and append binned color features, as well as histograms of color, to your HOG feature vector. 
* Note: for those first two steps don't forget to normalize your features and randomize a selection for training and testing.
* Implement a sliding-window technique and use your trained classifier to search for vehicles in images.
* Run your pipeline on a video stream (start with the test_video.mp4 and later implement on full project_video.mp4) and create a heat map of recurring detections frame by frame to reject outliers and follow detected vehicles.
* Estimate a bounding box for vehicles detected.

## [Rubric](https://review.udacity.com/#!/rubrics/513/view) Points
###Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
###Writeup / README

####1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Vehicle-Detection/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

###Histogram of Oriented Gradients (HOG)

####1. Explain how (and identify where in your code) you extracted HOG features from the training images.

The function which extracts HOG features is defined in the second cell: get_hog_features. It calls skimage's hog function. It is called in the extract_features function, a bit later. That function changes the color space and then concatenates the spacial, histogram, and hog features together, based on the specified parameters. The same thing is done in single_img_feautures, which is the next function. 

Those functions are used to extract the training features in the fourth cell. Later, when I run my sliding window search, I use the single_img_features fuction to retrieve just one windows's hog features.

An example of a the hog image of a car and the hog image of not a car are give in hog_car_image_0/1/2.jpg and hog_notcar_image_0/1/2.jpg

####2. Explain how you settled on your final choice of HOG parameters.

I tried various combinations of parameters and after a few trials, I found that color_space = 'HLS', spatial_size = (32,32), hist_bins = 32, orient = 9, pix_per_cell = 8, cell_per_block = 2, hog_channel='ALL' gave good results. You can see an example hog_image in hog_car_image_0/1/2.jpg and hog_notcar_image_0/1/2.jpg. Basically, I was looking at the larger image and trying to make sure that there was a noticable difference between the areas with and without cars. Really, this step was more art than science, in the end.

####3. Describe how (and identify where in your code) you trained a classifier using your selected HOG features (and color features if you used them).

The classifier runs in the 6th Cell. I trained a linear SVM with a very low C value, 0.0001, to reduce overfiting. I did use both HOG and color features to get an accuracy in the test set of 99%, and I was happy with that. That being said, I'm not sure if this is sufficent; I don't really check Kappa, precision, or recall, and those statistics would be more telling than just a simple accuracy metric. For an example of my feature vector, scaled and unscaled, please see "plot_feature.png"



###Sliding Window Search

####1. Describe how (and identify where in your code) you implemented a sliding window search.  How did you decide what scales to search and how much to overlap windows?

I used three sizes of window: 64, 96, and 112, and I only searched roughly the bottom half of the image, between y values of 360 and 630. This is shown in the get_windows function, cell 16. Please also refer to the images, small_windows.jpg, med_windows.jpg, and big_windows.jpg for examples. 

####2. Show some examples of test images to demonstrate how your pipeline is working.  What did you do to optimize the performance of your classifier?

In order to check to see tha tmy pipeline was working, I tested the classifier on the windows I just drew. This is done with the search_windows function. The output is shown in small_windows_cars.jpg, med_windows_cars.jpg, and big_windows_cars.jpg. I was happy with the accuracty my classifier achieved.
---

### Video Implementation

####1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (somewhat wobbly or unstable bounding boxes are ok as long as you are identifying the vehicles most of the time with minimal false positives.)

My videos are available in the github at "test_video_output.mpeg" and "project_video_output.mpeg"

####2. Describe how (and identify where in your code) you implemented some kind of filter for false positives and some method for combining overlapping bounding boxes.

I recorded the positions of positive detections in each frame of the video.  From the positive detections I created a heatmap and then thresholded that map to identify vehicle positions. In get_heatmap(), I required a minimum of three false positives to be reported, as well as at least 5% of the total detections. I then used `scipy.ndimage.measurements.label()` to identify individual blobs in the heatmap. As opposed to assuming each blob corresponded to a vehicle, I further filtered the heatmap in the function filter_labels. I only reported labels with dimension larger than 10x10, and with a mean squared certainty greater thatn 5%, and a maximum certain greater than 10%.

You can see my pipeline in the following four images for test image 1:
"final_windows.jpg" shows the windows which have been labeled as "car"
"final_heatmap.jpg" shows the heatmap generated from those windows
"final_label.jpg" is the unfiltered labels from the scikit label function
"final_output.jpg" is the filtered labels drawn on the image.

This does not show the averageing which I also do in the video. Each frame is a wieghted average of alpha times the current frame plus (1-alpha) times the previous frame. I found alpha = 0.8 gave greater stability in the video tests.


---

###Discussion

####1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

I have both false positive and false negatives in my implementation. It was very hard to find a good threshold value that would filter out false positives that would not also filter out actual cars! Notably, I found that color mattered a lot. Even in "final_windows.jpg", it is clear that my classifier is incredibly confident that the black car is a car, and less confident that the white car is a car. I think this is due to in some part to the fact that the white car looks awfully similar to the sky! (There are sky pictures in the notcar data set). I did some resarch, and found that I'm not the only one with this problem. Tesla had a crash where the car thought a large semi truck was just the sky: [https://www.nytimes.com/2017/01/19/business/tesla-model-s-autopilot-fatal-crash.html?_r=0]. Another issue I have is that shadows and trees and things can easily be miscontrued for some part of a car.

Response to First Submission Review:
Chaning the color space from HLS to YCrCb prompted a huge increase in the final box stability, even when omitting the color histogram. There's a few places still where I get a false positives, but they're quite few, and I successfully track the vehicles the entire time. Thanks for the tip.