
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

[image1]: ./figures/undistortion.png "Undistorted"
[image2]: ./figures/undistorted_img.png "Road Transformed"
[image3]: ./figures/threshold.png "Threshold Example"
[image4]: ./figures/region2.png "Warp Example"
[image5]: ./figures/persp_fit.png "Fit Visual"
[image6]: ./figures/pipeline.png "Output"
[video1]: ./out_project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the fourth (4th) code cell of the IPython notebook located in "./laneFinder.ipynb".  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image (thresholding steps at fifth (5th) code cell).  Here's an example of my output for this step.

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `topDownPerspective()`, which appears in the seventh (7th) code cell.  The `topDownPerspective()` function takes as inputs an image (`img`), as well as source (`src_pts`) and destination (`dst_pts`) points.  I chose to create the source and destination points in the following manner (Please note, `verts` was used in an earlier function and was equally viable for `src_pts` :

```python
#Defined Region coordinates
TOP_LEFT_XY = [.44,.65]
TOP_RIGHT_XY = [.56,.65]
BOTTOM_LEFT_XY = [.20,.92]
BOTTOM_RIGHT_XY = [.80,.92]

#Offsets for perspective transform
w_offset = 300
h_offset = 50

v1 = np.multiply(np.flip(tImage.shape[:2], axis=0), BOTTOM_LEFT_XY)  #Bottom Left
v2 = np.multiply(np.flip(tImage.shape[:2], axis=0), TOP_LEFT_XY)     #Top Left 
v3 = np.multiply(np.flip(tImage.shape[:2], axis=0), TOP_RIGHT_XY)    #Top Right
v4 = np.multiply(np.flip(tImage.shape[:2], axis=0), BOTTOM_RIGHT_XY) #Bottom Right

verts = np.array([v1,v2,v3,v4], dtype=np.int32)

src_pts = np.float32(verts)

dst_tl = [w_offset,h_offset]
dst_tr = [shpx-w_offset,h_offset]
dst_bl = [w_offset,shpy - h_offset]
dst_br = [shpx-w_offset,shpy-h_offset]
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 256, 662      | 300, 670        | 
| 563, 468      | 300, 50      |
| 716, 468     | 980, 50      |
| 1024, 662      | 980, 670        |

I verified that my perspective transform was working as expected by drawing the `src_pts` and `dst_pts` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

In the eighth (8th) code block I created `identifyLanes`.  I take a histogram for the bottom half of the image; further we find the peaks for the left and right halves of the remainder image (Peaks meaning hot pixels that exist after thresholding).  After defining window sizes, starting from the bottom-up, I find nonzero pixels within these windows, append them to a list for later, and cascade upwards in a loop with the windows, reaching for the greater concentration of nonzero pixels.  The lists are used to define left and right lanes, which are then fit to a 2nd order polynomial:

![alt text][image5]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

In the tenth (10th) code block I created `findCurvatureOfLane`.  Arguments are the left and right lane x values, with associated y values that were fit with the polynomial from earlier.  Using a liberal sense of rounding, we define a constant for meters/pixel; fit a curve with the new scale; and finally calculate the curvature of each lane.

In the eleventh (11th) code block I created `findDevFromCenter`.  Arguments include left and right lane x values with associated y values, vehicles position relative to image x size, and image height.  Using the image height I find the x value for the region closest to the vehicle.  Using the vehicle's current position, I find the difference between the vehicle and center of the lane's region.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in the twelfth (12th) code block in the function `showLane()`.  Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./out_project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Rapid road changes (improvements/damage) can skew my lanes to the point of being unusable, I find this can be solved with designing the system to include government regulations, and to have the visuals/detection methods include their parameters in decision making.

Steep curves that can hide lane lines from the vehicles will have the current system try to find lanes where lanes would not exist.  This can be alleviated with future predictions for lane path; which can use previous information and visible curvature information to predict the upcoming curve.  This can help future lanefinding while not entirely relying on the visual lanes within camera sight.  Additionally cameras on the side (if possible) could assist with the situation.
