
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


[image1]: ./output_images/undistortion.png "Undistorted Chessboard"
[image2]: ./output_image/undistortion_lane.png "Undistored Road"
[image3]: ./output_images/gradient_x.png "Gradient X"
[image4]: ./output_images/gradient_y.png "Gradient Y"
[image5]: ./output_images/gradient_mag.png "Gradient Magnitude"
[image6]: ./output_images/gradient_direction.png "Gradient Direction"
[image7]: ./output_images/s_thresholded.png "S Thresholded"
[image8]: ./output_images/h_thresholded.png "H Thresholded"
[image9]: ./output_images/l_thresholded.png "L Thresholded"
[image10]: ./output_images/r_thresholded.png "R Thresholded"
[image11]: ./output_images/combined_output.png "Combined Output"
[image12]: ./output_images/perspective_transform.png "Perspective Transform"
[image13]: ./output_images/histogram.png "Histogram"
[image14]: ./output_images/sliding_window_search.png "Sliding Window Search"
[image15]: ./output_images/detected_lane_on_original.png "Detected Lane on Image"



## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used combination of color and gradient thresholds to come up with final image in which lane were detected nicely. 

The gradient along X and Y direction, magnitude of gradient along X and Y and direction gradient all these were combined together. The Hue color threshold is not sensitive to shadows so a mask of it was applied to the combination to remove shadows. A mask of Lightness threshold was also applied to remove very dark pixels.

Saturation threshold was calculated since it work nicely with yellow lane detection. The Saturation threshold was combined with Red Threshold. Red threshold was used since it works nicely with white lane pixels. Again Hue threshold mask was applied to remove shadows.


Gradient X
![alt text][image3]

Gradient Y
![alt text][image4]

Gradient Magnitude
![alt text][image5]

Gradient Direction
![alt text][image6]

S Thresholded
![alt text][image7]

H Thresholded
![alt text][image8]

L Thresholded
![alt text][image9]

R Thresholded
![alt text][image10]

Combined Color and Gradient
![alt text][image11]


#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `perspective_transform`. 
The function takes a img as input. Calculates a matrix from source and destination points with function `cv2.getPerspectiveTransform()` and applies it to the image to returned birds eye view with function `cv2.warpPerspective()`

Following source and destination points were used:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 220, 720      | 320, 720        | 
| 1100, 720      | 920, 720      |
| 570, 470     | 320, 1      |
| 722, 470      | 920, 1        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image12]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

First i calculated histogram of the lower half of image. The histogram resulted in two peaks which give a point in x-direction where the left and right lanes are starting.

![alt text][image13]

Then i applied sliding window search algorithm to trace out all the pixels which belongs to lane lines. The detected left and right lane pixels was used to fit a polynomial equation of power 2. This fitted polynomial corresponds to the equation of left and right line. 

From the next frame onwards these fitted equations was used to detect lane lines.

![alt text][image14]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The function `cal_radius_of_cuvature()` is used to calculate the radius of curvature.

Now that we know lane equation we will use it to calculate radius of curvature. Since the lane equation is polynomial of degree 2 there is a equation to calcuate radius of curvature. We will calculate radius of curvature at bottom of image. For that we will generate all the y points and corresponding x points through equation. Then we will covert the x and y points from pixels to meters. Finally we will use the function of radius of curvature calculation fo calculte it.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this in function `visualizer_lane_on_original_image()`

![alt text][image15]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./output_videos/project_video.mp4)
Here's a [link to my challenge video result](./output_videos/challenge_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Although i tried to address as many issues as possible the biggest challenge of detecting what lane curvature value are bad wrt to past calculations and what line fits are bad wrt to lane curvature. I used debugging methods to find values which works best. We are assuming that the road is flat but in real world it wont be the case and our perspective transform might fail. Also if there are lots of turns on road that the chances of algorithm not working is high and even after doing perspective transform we might include lots of noisy pixels.