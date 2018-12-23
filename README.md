##Advanced Lane Finding Project
### Writeup

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

[image1]: ./Images/calibration1_undistorted.jpg "Undistorted"
[image2]: ./Images/straight_lines1_undistorted.jpg "Road undistorted"
[image3]: ./Images/Thresholds_binary.png "Threshold Binary Example"
[image4]: ./Images/Perspective_transform.png "Perspective Transform Example"
[image5]: ./Images/Histogram.png "Histogram"
[image6]: ./Images/Sliding_Windows.png "Sliding Windows"
[image7]: ./Images/Polynomial_fit.png "Polynomial fit on image"
[image8]: ./Images/Shading_lane_area.png "Shading lane area"
[image9]: ./Images/Rad_curve_formula.png "Formula"
[image10]: ./Images/Lane_boundaries.png "Lane boundaries Example"
[image11]: ./Images/Visual_display.png "Visual display Example"

[video1]: ./Final_Documents_ALD/project_solved.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
I have first applied computer vision techniques on the test images to perform advanced lane detection. Then I have used the same techniques in a pipeline for lane detection on video.

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the second and third code cell of the IPython notebook located `some_file.py`).  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `obj_points` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `img_points` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

OpenCV functions findChessboardCorners() and drawChessboardCorners() was used to find and draw corners in an image of a chessboard pattern. I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test images and chessboard images using the `cv2.undistort()` function and obtained this result: 

Chessboard Image

![alt text][image1]


### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I apply the distortion correction to one of the test images like this one:

![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used a combination of color and gradient thresholds to generate a binary image. 

First I visualised the RGB and HLS color channels for the test image. Selected suitable channels for suitable thresholds. I selected L channel for applying sobelx operator and then binary image output.Then I separately performed color thresholds for yellow lane on RGB image and HLS image and, for white lane on RGB image.

(Code cell 5 & 6) 
  
  A combined binary image of the color threshold and sobelx threshold was done. Here's an example of my output for the operations.

![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code (code cell 7) for my perspective transform includes OpencV functions called `getPerspectiveTransform()` `warpPerspective()`.The `getPerspectiveTransform()` function takes as inputs, source (`src`) and destination (`dst`) points and saves the transformation matrix.  I chose the the source and destination points with respect to the image shape in the following manner:

```
row, col, _ = img.shape
src_points = np.float32([[0.2*col,0.9*row],
                         [0.45*col,0.64*row],
                         [0.55*col,0.64*row],
                         [0.8*col,0.9*row]])
dst_points = np.float32([[0.2*col,0.9*row],
                         [0.2*col,0*row],
                         [0.8*col,0*row],
                         [0.8*col,0.9*row]])
```
                         
Which are actually the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 270, 674      | 270, 674       | 
| 579, 460      | 270, 0      |
| 720, 460     | 1035, 0      |
| 1035, 674      | 1035, 674      


`warpPerspective()` in the `transform_n_warp()`  takes the image and pts as input to give the perspective transformed image. 
I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped binary image.

![alt text][image4]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

Histogram (code cell 10) was used to detect the lane regions in the perspective transformed image as shown in the below image:

![alt text][image5]

 Next step was to implement sliding windows method. The (code cell 12) with comments explains the algorithm. The below image shows the sliding windows and the the polynomial fit.
 
 ![alt text][image6]
 
 ![alt text][image7]
 
 The sliding window appproach to fit polynomial can be applied to the first frame of a video frame. Once the line region is identified we can skip the sliding windows step (code cell 16,17,18,19).  
 
Shaded lane region

![alt text][image8]

I used the Udacity provided code to get desired outputs.
  
#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The radius of curvature is calculated in the code cell 20 using the below formula

![alt text][image9]

The lane boundaries are drawn and warped back using OpencV function `warpPerspective()` (inverse perspective transform) in code cell 21. The below image shows the result.

![alt text][image10] 

The visual display fo left and right curvatures and the calculated value of the distance the car is off from the centre is done in code cell 22.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

The steps for visual display in code cell 22 `display_on_frame()` outputs the following image:

![alt text][image11]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

A class called `Lane()`is created for encompassing the pipeline for lane detection on video stream from code cell 23. 

Here's a [link to my video result](https://github.com/tomjose1792/CarND-Advanced-Lane-Lines/blob/master/project_solved.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

I found difficulty initially in understanding the algorithm for fitting the 2nd order polynomial using sliding windows approach and later the algorithm for finding the lanes skipping sliding window approach from Udacity tutorials. 
The other issue was finding the right source points for the perspective transformation for which I had to consider trial and error. Though the final pts worked for the project video, did not for the challenge videos. Probably for hard curves dynamically changing algorithm for finding the source points might have to be done.
Another issue was selecting the right thresholding methods and its parameters. Herein the project I have used simple thresholding methods. It can fail in different lighting conditions like if the lane color changes in a different lighting environment. Therefore in those conditions we might have to be more careful in selecting the thresholding methods for the binary output image.