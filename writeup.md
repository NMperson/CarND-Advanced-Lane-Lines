##Writeup Template
###You can use this file as a template for your writeup if you want to submit it as a markdown file, but feel free to use some other method and submit a pdf if you prefer.

---

**Advanced Lane Finding Project**

The goals / steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

### For the code portion of this submission, the ideas are checked in P4.ipynb, but Submission.py should stand alone by itself. Code will exist in both places, but I will only refer to the python file directly in this writeup

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points
###Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
###Writeup / README

####1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point. 

You're reading it!
###Camera Calibration

####1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this is split into multiple locations. First, I read in the calibration images and calculated the object points and image points. This is done one lines 524 and 525.

Then, line 12 is where the object points and image points are prepared. I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection. 

That is only done one time, and objpoints and imgpoints are saved globally. Then, the actually calibration and undistortion is done as part of the image pipeline. See line 462, which calls undistort, line 37. I used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained the result shown in "image_undist.jpg"

###Pipeline (single images)

####1. Provide an example of a distortion-corrected image.
The examples in this writeup will use "image.jpg" as the starting point.
"image_undist.jpg" is an example of a distortion corrected image

####2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.
I used a combination of color and gradient thresholds to generate a binary image. Line 466 calls the pipeline, which is then thresholded on line 470. The actual pipeline function starts at line 107. There are two steps to this funciton. First, I do some color analysis, then gradient analysis. For the colors, I select the R and G channel, then convert to HLS and take the L and S channels. Each channel is thresholded indivudually and summed into a color threshold.

Then, I take the grad in the x and y directions, and calculate the magnitude and direction of the gradient. Thresholding is applied to each gradient based on what I've found will do a good job of detecting lane lines, then the gradients are summed.

The colors and gradients are then summed together - and colors are weighted three times that of the gradient to give better results. Then, this summed image is thresholded on line 470.

"image_thresh.jpg" shows my thresholded image. "image_comb.jpg" shows the pipeline image out of the first step. Also of interest should be "image_all_colors.jpg", and "image_all_grad.jpg", which show various steps of the pipeline.

####3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

Again, I do this is two steps. First, calcwarp is used to cal getPerspectiveTransform and calculate the warping matrix and the inverse warping matrix. This function is defined starting on line 155.

The dest vertices were chosen to create a 300x300 image.

The source verices were chosen by trial and error. The y coordinates were chosen to crop roughly to bottom 10% and top 60%. Then the x coordinates are chose by different heuristics for the top and bottom. The bottom x coordinates just use the extent of the image. The top x coordinates us the center of the image, plus or minus some percentage, xtp. The values for xtp was determined by making sure straight lane lines appeared straight on a warped image.

After this has all been done, the warp() function, defined on line 188, can be called. This function takes the result from calcwarp to do the actual perpective warp

"image_warped.jpg" shows and example of a warped and thresholded image

####4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

My lanelines we calculated in two steps. First, I use a variation of the histogram method discussed in class to get a baseline. I split the image into two halves to find the left and right lanes, but instead of taking the index of the max, I use a weighted average. This should make my method more robust, especially in the case of outliers. This method is in the code on page 194.

On line 478, the histogram method is called to get the points. These points are only used if my sanity checking thinks the previous fit is either unitialized or in some way bad.

Finally, on lines 491 and 492, I use a polynomial method to find lane lines. The method uses either the polynomial values from the previous frame or the histogram method, depending.

That method is defined on ine 228. What this function does is it uses the previous polynomial method and grabs the pixels plus or minus some distance from that line. It saves any points it finds, then once it has a sufficiently large list of points, it begins to use the current poly fit line to grab additional pixels with even greater accuracy. Additionally, I want to note the the distance from the line decreases from the bottom to top of the image. The increases the chance I'll find the lane and decreases the chance the alogrithm will find unwanted pixels at the top.

Please refer the following images for the polynomial fit:
"image_l_fit.jpg" shows the pixels used to fit the left lane line
"image_r_fit.jpg" shows the pixels used to fit the right lane line
"image_overlay.jpg" shows the left and right lane lines, in blue and red, and the lane in green

####5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

Lines 509 and 510 are where I call the functions, but the core code is at lines 391 and 417, respectively. 

Lane 391 shows calcCurve, which takes a polynomial and meter to pixel conversion constants provided by Udacity. A list of values is created 1 through 300, from which x values are calculated. Then, those two lists are converted from pixel dimensions into meters dimension. A polyfit is run on the meter dimension lists, then the function (1+(2*A*y+B)**2)**(3/2)/abs(2*A), also provided by Udacity is used to calculate the curvature. The left and right cuvature values are averaged together on line 411.

For calculating the left/right, line 417, the only constant I needed was the X meter to pixel conversion constant. All that I needed to do here was calculate the X position of the left and right lane lines at the base of the image and take the median. Then, I can compare that to the center of my image, which gives me the offset in terms of pixels, then I can conver into meters useing the constant.

"image_text.jpg" shows the text at the top of the image.

####6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in lines 501 and 505 in my code in `submission.py`. For warp, I just use the inverse of the matrix described earlier, and I superimpose the lanes onto the image using the same technique we used in example one.

"image_overlay_unwarped.jpg" shows just the lane lines

"image_composite.jpg" shows the lane lines on top of the original image.

---

###Pipeline (video)

####1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's my submission: "project_video_output.mp4"

---

###Discussion

####1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

In the end, I'm not really in love with the lane line fit algorithm I came up with. I think it is a little more elegant in the code than the one Udacity proposed, but my lane lines are definitely a little less stable.

I'll also say that I could spend weeks fine tuning the thresholding pipeline. Increasing or decreasing the thresholds and the weights of each parameter. I know the eight I've chosen will get me where I want to go - but I know I'm still over relying on the color information, and that does cause me some issues once I get into the shadows, for example.

