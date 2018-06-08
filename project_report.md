## Report


**Vehicle Detection Project**

The goals / steps of this project are the following:

* Perform a Histogram of Oriented Gradients (HOG) feature extraction on a labeled training set of images and train a classifier Linear SVM classifier
* Optionally, you can also apply a color transform and append binned color features, as well as histograms of color, to your HOG feature vector. 
* Note: for those first two steps don't forget to normalize your features and randomize a selection for training and testing.
* Implement a sliding-window technique and use your trained classifier to search for vehicles in images.
* Run your pipeline on a video stream (start with the test_video.mp4 and later implement on full project_video.mp4) and create a heat map of recurring detections frame by frame to reject outliers and follow detected vehicles.
* Estimate a bounding box for vehicles detected.

[//]: # (Image References)
[image1]: ./figures/classes.png
[image2]: ./figures/hog_extract.png
[image3]: ./figures/bounding_boxes.png
[image4]: ./figures/pipeline.png
[image5]: ./figures/heatmaps.png
[video1]: ./project_video.mp4

## [Rubric](https://review.udacity.com/#!/rubrics/513/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Vehicle-Detection/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Histogram of Oriented Gradients (HOG)

#### 1. Explain how (and identify where in your code) you extracted HOG features from the training images.

The code for this step is contained in code block 12 cell of the IPython notebook.

I started by reading in all the `vehicle` and `non-vehicle` images.  Here is an example of one of each of the `vehicle` and `non-vehicle` classes:

![alt text][image1]

I then explored different color spaces and different `skimage.hog()` parameters (`orientations`, `pixels_per_cell`, and `cells_per_block`).  I grabbed random images from each of the two classes and displayed them to get a feel for what the `skimage.hog()` output looks like.

Here is an example using the `Grayscale` color space and HOG parameters of `orientations=9`, `pixels_per_cell=(8, 8)` and `cells_per_block=(2, 2)`:


![alt text][image2]

#### 2. Explain how you settled on your final choice of HOG parameters.

I tried various combinations of parameters and processed through the images, comparing examples with still images and video images.  I generally discovered that the parameters originally used were effective enough; `orientations = 9`, `pixels per cell = (8,8)`, `cells per block = (2,2)`.  I found that altering the colour space gave the largest changes in the end result.  I have experimented with multiple colour spaces; but finally settled on `color_space = 'YCrCb'`.

#### 3. Describe how (and identify where in your code) you trained a classifier using your selected HOG features (and color features if you used them).

I trained a linear SVM on code block 17.

I trained the linear SVM with hog features, spatial features, and colour histogram features.  HOG (code block 12) parameters are explained above; the spatial features (code block 14) is a 1D array of the image separated into each colour channel; the colour histogram features (code block 13) are an assortment of bins for each colour channel separately.

### Sliding Window Search

#### 1. Describe how (and identify where in your code) you implemented a sliding window search.  How did you decide what scales to search and how much to overlap windows?

The implementation exists in code block 21, around lines 30-34 defines the windows parameters.  This process is used since the hog features of the area of interest is pre-processed as to prevent unnecessary calculations later in the pipeline.  The windows are aligned with the hog cells, so that we do not need to invoke the hog feature extraction for every window (taking a subset of the previously processed data).

Further below I loop through each window by `nxsteps` and `nysteps`.  Each loop processes through the feature extraction and prediction.

![alt text][image3]

#### 2. Show some examples of test images to demonstrate how your pipeline is working.  What did you do to optimize the performance of your classifier?

Ultimately I searched on two scales using `YCrCb` 3-channel HOG features plus spatially binned color and histograms of color in the feature vector, which provided a nice result.  Here are some example images:

![alt text][image4]
---

### Video Implementation

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (somewhat wobbly or unstable bounding boxes are ok as long as you are identifying the vehicles most of the time with minimal false positives.)
Here's a [link to my video result](./out_project_video.mp4)


#### 2. Describe how (and identify where in your code) you implemented some kind of filter for false positives and some method for combining overlapping bounding boxes.

I recorded the positions of positive detections in each frame of the video.  From the positive detections I created a heatmap and then thresholded that map to identify vehicle positions.  I then used `scipy.ndimage.measurements.label()` to identify individual blobs in the heatmap.  I then assumed each blob corresponded to a vehicle.  I constructed bounding boxes to cover the area of each blob detected.  

Here's an example result showing the heatmap from a series of frames of video, the result of `scipy.ndimage.measurements.label()` and the bounding boxes then overlaid on the last frame of video:

### Here are six frames and their corresponding heatmaps, overlapping bounding boxes, and the resulting bounding box for the final frame:

![alt text][image5]

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Once I reached the end of the initial pipeline design; I aimed to adjust a few features in two stages.
Stage 1
* Adjusting my trained model until I found a result that can be manipulated appropriately
This involved changing my input parameters, mostly the color space, and adjusting + adding additional training data for the SVM.  I mostly looked for a strong result where the bounding boxes overlapped my target and allowed me to threshold away the false positives with ease.

Stage 2
* Adjusting my heatmap thresholding strategy to remove false positives and strengthen the overlapping bounding boxes of my targets
This involved creating a rolling average of the current frame + previous 29 frames; then filter through a threshold as to remove spontaneous positives, but will keep strong contenders within the heatmap.

I faced many issues when tuning the parameters, as each parameter can have a very visible, or very non-visible affect on the total pipeline.  I was only able to focus on the current still images, but I do not know how this would affect video and rolling average calculations.

Working with the heatmap thresholding was very trial and error heavy, as testing could bring unexpected behaviours during the video.

With my current implementation, there will surely be difficulties with opposing traffic when there is not a barrier in-between the roads.  Additionally any images of vehicles (billboards, etc) may cause false positives that will persist through thresholding.  Additionally vehicles close to other vehicles may indicate only one vehicle exists, when really it is the heatmaps combining and creating a single bounding box.

