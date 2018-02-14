## Writeup

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

[//]: # "Image References"
[calibrate]: ./writeup_images/calibrate.png "Calibrate"
[original]: ./writeup_images/original.png "Original"
[undistort]: ./writeup_images/undistort.png "Undistort"
[warp]: ./writeup_images/warp.png "Warp"
[binerize]: ./writeup_images/binerize.png "Binerize"
[denoise]: ./writeup_images/denoise.png "Denoise"
[crop]: ./writeup_images/crop.png "Crop"
[hist]: ./writeup_images/hist.png "Histogram"
[filter]: ./writeup_images/filter.png "Filter"
[poly]: ./writeup_images/poly.png "Poly"
[drawn]: ./writeup_images/drawn.png "Drawn"
[unwarp]: ./writeup_images/unwarp.png "Unwarp"
[pipeline]: ./writeup_images/pipeline.png "Pipeline"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  Note: All Code is located in `./example.ipynb`  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The calibration was done by loading in all of the images found in `./camera_cal/`, then iterating over each one and using `cv2.findChessboardCorners()` to locate the chessboard corners.  If corners were found, I added the corners and the grid to lists `imgpoints` and `objpoints` respectively.

Then, using the `objpoints` and `imgpoints` lists from before and the `cv2.calibrateCamera()` function I created a camera matrix (`mtx`) and distortion coefficients (`dist`).  After that, I created a `undistort` function that wrapped `cv2.undistort()` and used camera matrix and distortion coefficients from before.  Here's an example of the undistort function:

![Calibrate][calibrate]

### Pipeline (single images)

To start with, I loaded in an image from `./test_images/` to use as my original image:

![original][]

#### 1. Provide an example of a distortion-corrected image.

_This step is located in the `undistort()` function of the notebook._

The first step in the pipeline was to apply the `undistort` function I created before on the original image, here's the output:

![undistort][]

The differences are subtle, but if you look closely at the edges you will notice this did undistort the image.

#### 2. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

_This step is located in the `warp()` function of the notebook._

The next step I did was to perform a prospective warp the image so that we can get a birds eye view to make finding the lane lines easier.

![warp][]



Note: I did this before the thresholding since I didn't want to distort the output of the threshold.

#### 3. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

_This step is located in the `binerize()` function of the notebook._

I used a combination of color and gradient thresholds and I tried using blurs as well in order to reduce some of the noise in the gradient thresholds, but this had little effect so in the end I scrapped it.

![binerize][]

#### 



#### 4. (Not in Rubric) Reducing noise

_This step is located in the `reduce_noise()` function of the notebook._

In the image above you can see how there's quite a bit of noise that's just pixels sitting all by themselves.  I figured this could be easily removed, I looked around eventually came to use a combination of the `cv2.getStructuringElement()`  function and `cv2.morphologyEx()` functions.

![denoise][]

#### 5. (Not in Rubric) Cropping

_This step is located in the `crop()` function of the notebook._

If we assume that the car is generally in the center of the lane, we know generally where the lanes start at the bottom of the image, and, more importantly we know where it doesn't start!  So, in order to make it easier on the histogram I trimmed the bottom right and bottom left corners off, as shown here:

![crop][]

#### 



#### 6. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

_This step is located in the `find_lines()` & `find_lines_from()` functions of the notebook._

To find the lane lines I used the filter method from Udacity, where I start by taking a histogram of the bottom half of the image:

![hist][]

starting from the bottom, iteratively slide a filter up the image looking for pixels & re adjusting to center on them.  (Left colored in blue, right colored in red):

![filter][]

Lastly, to find the polynomial I used the `np.polyfit()` function on the pixels of each lane, here's the polygons drawn on the last image:

![poly][]

#### 5. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

_This step is located in the `draw_lane()` & `unwarp()` functions of the notebook._

To draw the lane back on the original image, I first filled in the area between the two polynomials found in the last step:

![drawn][]

Then, applied the reverse of the `warp()` function from before and pasted it on top of the undistorted original image:

![unwarp][]



#### 6. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

_This step is located in the `calculate_off_center()` & `calculate_curvature()` functions of the notebook._

To calculate the distance from the center, I took the average of the x location of the polynomials at the bottom of the screen and compared it to the center of the image.  This does assume that the camera is centered on the car.

To calculate the curvature, I used the function given to us by Udacity, then averaged the right & left curvatures.

#### 7. (Not in Rubric) Overlay the curvature and distance from center on the image.

_This step is located in the `pipeline()` function of the notebook near the bottom._

I pretty much just used the `cv2.putText()` function, there wasn't much to it.  But I have a picture of it so this step is just a lame excuse to use the picture:

![pipeline][]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./output_video.mp4) and Here's a [link to the challenge video](./output_harder_challenge_video.mp4)  that I spent 0 time on but ran it through my pipeline just for funzies.

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

My biggest issue was understanding the parts where Udacity kinda gave the answer away.  Like calculating the curvature, truthfully that's kinda a black box, I'm not sure what it does.  Also, I really felt like Deep Learning needed to be applied here and that I was doing work that didn't need to be done.
