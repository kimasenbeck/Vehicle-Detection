**Vehicle Detection Project**

The goals / steps of this project are the following:

* Perform a Histogram of Oriented Gradients (HOG) feature extraction on a labeled training set of images and train a classifier Linear SVM classifier
* Optionally, you can also apply a color transform and append binned color features, as well as histograms of color, to your HOG feature vector. 
* Note: for those first two steps don't forget to normalize your features and randomize a selection for training and testing.
* Implement a sliding-window technique and use your trained classifier to search for vehicles in images.
* Run your pipeline on a video stream (start with the test_video.mp4 and later implement on full project_video.mp4) and create a heat map of recurring detections frame by frame to reject outliers and follow detected vehicles.
* Estimate a bounding box for vehicles detected.

[//]: # (Image References)
[image1]: ./output_images/vehicle.png
[image2]: ./output_images/non_vehicle.png
[image3]: ./output_images/hog_example5.png
[image4]: ./output_images/sliding_windows.jpg
[image5]: ./output_images/heatmap_images.jpg
[image6]: ./output_images/multiple.png
[image7]: ./output_images/single.png
[video1]: ./test.mp4

## [Rubric](https://review.udacity.com/#!/rubrics/513/view) Points
### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.

You're reading it!

### Histogram of Oriented Gradients (HOG)

#### 1. Explain how (and identify where in your code) you extracted HOG features from the training images.

The code for this step is contained in the first through fourth code cells of the IPython notebook.  

I started by reading in all the `vehicle` and `non-vehicle` images.  Here is an example of one of each of the `vehicle` and `non-vehicle` classes:

![alt text][image1]
![alt text][image2]

I then explored different color spaces and different `skimage.hog()` parameters (`orientations`, `pixels_per_cell`, and `cells_per_block`).  I grabbed random images from each of the two classes and displayed them to get a feel for what the `skimage.hog()` output looks like.

Here is an example using the `RGB` color space and HOG parameters of `orientations=6`, `pixels_per_cell=(8, 8)` and `cells_per_block=(2, 2)`:


![alt text][image3]

#### 2. Explain how you settled on your final choice of HOG parameters.

I tried various combinations of parameters and ultimately found that using 3 color channels in the YCrCb color space resulted in the best outcome. Additionally, based on existing research on HOG parameters, I opted to set orientations to 9, since the research suggests that this is the optimal number of orientations. 

#### 3. Describe how (and identify where in your code) you trained a classifier using your selected HOG features (and color features if you used them).

I trained a linear SVM using SciKitLearn's LinearSVC function. You can find this code in cell 5 of my notebook. The data which I pass in to the SVC has been transformed using a StandardScaler in order to normalize all the features. 

### Sliding Window Search

#### 1. Describe how (and identify where in your code) you implemented a sliding window search.  How did you decide what scales to search and how much to overlap windows?

This part of my solution can be found in box 6 of my notebook. I implemented a sliding window search to find cars in the images. Initially, I used two different sizes of windows: (64, 64) and (128, 128). I figured that canning for windows of two different sizes would allow me to find cars both near and far from the horizon. However, it ended up giving me more false positives, so I opted to go with only the larger window size. I limited my search to the bottom half of the image, since it's not reasonable for cars to be located above the horizon line. Searching at a scale of 1.5 yielded the best results for me, with the fewest false positives. I opted to overlap by 50%, meaning that my window slid over by half a window size in each iteration.  

This image gives a visual indication of what the sliding windows look like. Note that this image does not actually correspond with the final parameters that I decided on, since I did not do any searching above the horizon line. However, this provides a good idea of what kinds of windows this algorithm searches for.

![alt text][image4]

#### 2. Show some examples of test images to demonstrate how your pipeline is working.  What did you do to optimize the performance of your classifier?

Ultimately I searched using the sliding window method using YCrCb 3-channel HOG features plus spatially binned color and histograms of color in the feature vector, which provided a nice result.  Here are some example images:

![alt text][image5]
---

### Video Implementation

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (somewhat wobbly or unstable bounding boxes are ok as long as you are identifying the vehicles most of the time with minimal false positives.)
Here's a [link to my video result](./test.mp4)


#### 2. Describe how (and identify where in your code) you implemented some kind of filter for false positives and some method for combining overlapping bounding boxes.

I recorded the positions of positive detections in each frame of the video.  From the positive detections I created a heatmap and then thresholded that map to identify vehicle positions.  I then used `scipy.ndimage.measurements.label()` to identify individual blobs in the heatmap.  I then assumed each blob corresponded to a vehicle.  Finally, I constructed bounding boxes to cover the area of each blob detected.  

This step was important, since without it, we'd have multiple boxes overlaid on top of each car. Here's an example image of what the bounding boxes look like before this step:

![alt text][image6]

For comparison, after constructing the heatmap and applying bounding boxes to the heatmap results, I get the following result:

![alt text][image7]


---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust? Talk about the approach you took, what techniques you used, what worked and why, where the pipeline might fail and how you might improve it if you were going to pursue this project further.  


This solution does reasonably well, but it could be improved. For one, the boxes are a bit jumpy between frames. With some smoothing steps, I could resolve this issue fairly simply. In addition, the pipeline doesn't do quite as well with cars near the horizon. I attempted to solve this issue by adding a second series of smaller windows to search over, but this didn't solve the problem entirely. Finally, I have not yet managed to fully prevent false positives. This can be a big issue, since a car might suddenly stop if it detects that a vehicle is in front of it. This is the sort of thing that can cause an accident in the real world, and merits a great deal of attention.

