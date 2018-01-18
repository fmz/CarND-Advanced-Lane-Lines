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

[distorted]: ./chess_original.jpg "Original"
[undistorted]: ./chess_undistorted.jpg "Undistorted"
[rd_dist]: ./road_original.jpg "Road Original"
[rd_undist]: ./road_undistorted.jpg "Road Undistorted"
[raw_road]: ./test_images/mytest1.jpg "Road Image"
[edges_road]: ./dump/edges_mytest1.jpg "Edges"
[orig_transform]: ./dump/horizontal_mytest1.jpg "Untransformed"
[virt_transform]: ./dump/vertical_mytest1.jpg "Transformed"

[warped]: ./dump/warped_mytest1.jpg "Warped edges"
[traced]: ./dump/traced_mytest1.jpg "Traced lane"
[out]: ./output_images/mytest1.jpg "Output"
[image6]: ./examples/example_output.jpg "Output"
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the second code cell of the IPython notebook located in "lanes.ipynb".

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

Original            |  Undistorted
:------------------:|:-------------------------:
![distorted]  		|  ![undistorted]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

Using the undistortion technique described in the previous section, here is an example of road images:


Original            |  Undistorted
:------------------:|:-------------------------:
![rd_dist]  			|  ![rd_undist]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image (see extract_lane_colors() and get_edges() in `lanes.ipynb`).  Here's an example of my output for this step.  (note: this is not actually from the original test images)

Raw Image           |  Edges
:------------------:|:-------------------------:
![raw_road]  			|  ![edges_road]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `vertical_perspective()` and `restore_perspective`, which appears under "perspective transform helpers" in `lanes.ipynb`.  The `vertical_perspective()` function takes as inputs an image (`img`) of a road, and transforms it to a bird's-eye view.  I chose to hardcode the source and destination points in the following manner:

```python
src_pts = np.float32([ \
		[shape[1]/13,shape[0]], \
		[4*shape[1]/9 - 5, 2*shape[0]/3 - 20], \
		[5*shape[1]/9 + 5, 2*shape[0]/3 - 20], \
		[12*shape[1]/13,shape[0]] \
       ])
dst_pts = np.float32([ \
		[offset, shape[0]], \
		[offset, 0], \
       [shape[1] - offset, 0], \
       [shape[1] - offset, shape[0]] \
       ])
```


I verified that my perspective transform was working as expected by drawing the `src` points onto a test image and verifying that the transformed points correspond to 'dst'

Untransformed       |  Bird's Eye
:------------------:|:-------------------------:
![orig_transform]  	|  ![virt_transform]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Now after getting the edges, and transforming them to a bird's eye view, I used the sliding window technique described in the lecture to extract, and separate out, the left and right line pixels (see: `detect_lane_lines()`).
Then, in `extrapolate_lane()`, I fit a polynomial over each of the 2 extracted lines.

Warped image        |  Extrapolated
:------------------:|:-------------------------:
![warped]  			|  ![traced]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I did this in `_get_radius()` in `lanes.ipynb`

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in lines # through # in my code in `yet_another_file.py` in the function `map_lane()`.  Here is an example of my result on a test image:

![alt text][out]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video_output.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The first issue I faced, which kept (and still is haunting me until now), is how to tune my sobel, saturation, and lighting thresholds properly to produce the most pristine lanes possible. This involved a lot of trial and error, which isn't ideal for a system that's meant to be robust.

Soon after, I was faced with the issue of having very few datapoints on one of the lanes to make my polynomial fit more reliable. I suppose I could keep a model where I continuously add more data points with time, but I just elected to take the average of the last 5 fits. Again, not ideal.

This system is also easy to fool. Imagine having a truck right in front of you with an ad that has vertical white and yellow lines. This system would detect those lines and treat them as lanes. The same could happen if a car drove closely enough on either of our sides, with prominent stripes on its side.

The main thing that bugs me here, is that this approach is certainly not reliable at all. I come from a country where lanes are, practically, optional; they could be faded, drawn over, non-existent, or simply ignored. There are a number of implicit rules as to how to act in these situations, which the car should emulate if it hopes to safely drive there.
