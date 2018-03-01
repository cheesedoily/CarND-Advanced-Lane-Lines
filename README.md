## Writeup Template

### You can use this file as a template for your writeup if you want to submit it as a markdown file, but feel free to use some other method and submit a pdf if you prefer.

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

[//]: # (Image References)

[image1]: ./output_images/undistort_camera_calib.png "Undistorted Camera Corners"
[image2]: ./output_images/undistort_lane_example.png "Undistorted Lane Example"
[image2]: ./output_images/binary_image_example.png "Binary Lane Image"
[image3]: ./output_images/binary_combo_example.png "Binary Example"
[image4]: ./output_images/perspective_transform.png "Perspective Example"
[image5]: ./output_images/mask_example.png "Mask Example"
[image6]: ./output_images/sliding_widnow.png "Sliding Window"
[image7]: ./output_images/previous_fit.png "Previous Fit"
[image8]: ./output_images/draw_lanes.png "Draw Lanes"
[video1]: ./output_images/output_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

Here you are

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the 2nd code cell of the IPython notebook located in "./examples/advanced.ipynb" 

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image. Thus, objp is just a replicated array of coordinates, and objpoints will be appended with a copy of it every time I successfully detect all chessboard corners in a test image. imgpoints will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.

I then used the output objpoints and imgpoints to compute the camera calibration and distortion coefficients using the cv2.calibrateCamera() function (via a convenience function `undistort()`). I applied this distortion correction to the test image using the cv2.undistort() function and obtained this result:


![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

The same `undistort` method is used to distortion-correct a sample lane line image (code found in the 4th code cell of the notebook)
![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I create a function in the 5th cell of the notebook called `make_binary()` which greates a binary image by using a combination of color, lightness and x-gradient (Sobelx) thresholds.

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warper()` and helper `warp_image()`, which appears in in the 3rd code cell of the notebook.  The `warper()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points, `warp_image()` takes the image the distortion matrix M (pre calculated).  I chose the hardcode the source and destination points in the following manner:

```python
src = [
        [194, undist.shape[0] - 1], 
        [1115, undist.shape[0] - 1], 
        [712, undist.shape[0] - 255],
        [570, undist.shape[0] - 255], 
      ]
base = 290
dst = [
        [base, undist.shape[0] - 1], 
        [1280 - base, undist.shape[0] - 1], 
        [1280 - base, 0],
        [base, 0], 
      ]
```

[[194, 719], [1115, 719], [712, 465], [570, 465]]
[[290, 719], [990, 719], [990, 0], [290, 0]]


This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 194, 719      | 290, 719        | 
| 1115, 719      | 990, 719      |
| 712, 465     | 990, 0      |
| 570, 465      | 290, 0        |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points (as a trapezoid) onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

First I mask the image such that potential noise of the bottom of the image is ignored (code block). This was especially important when using the sliding window technique (described below) to make sure that spurious bright spots at the bottom of the image didn't affect the first search window.

![alt text][image5]

Then I define two functions `sliding_window()` and `fit_using_prev()`. Sliding window just takes in the binary warped imaged while fit using previous needs the previous fit polynomial to begin its search.

`sliding_window` works by "stairstepping" up from the bottom of the image looks only withing some margin of the previous horizontal slice of the image. `fit_using_prev` works by using the previous polynomial and only searches with an x margin from there.

![alt text][image6]
![alt text][image7]


#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

Radius of curvature and offset are calculated in the funciton `calculate_curvature_and_offset()`. In order to get the radius is physical space (i.e. km) i first have to us the polynomial `fit_*_cr` variables to calculate the radius.

```
    left_curverad = ((1 + (2*left_fit_cr[0]*y_eval*ym_per_pix + left_fit_cr[1])**2)**1.5) / np.absolute(2*left_fit_cr[0])
```

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in the function `draw_lane()`.

![alt text][image8]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./output_images/output_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The lane lines wobble a decent amount. I noticed the biggest issue is for frames where there is no lane line at the the bottom of the image. 

In order to combat this, i would keep more state of previous frames to increase the cloud of potential lane line points.

Additionally any noise in the binary image at the bottem of the image really hurt the sliding window approach. Masking helped a little, but is fairly naive.