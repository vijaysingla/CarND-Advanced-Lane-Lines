

## Self Driving Car Engineer NanoDegree Program

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


[image1]: ./output_images/CameraCalibration.png

[image2]: ./output_images/Color_and_gradient_thresholding.png

[image3]: ./output_images/Perspective_Transformation.png

[image4]: ./output_images/Undistortion.png

[image5]: ./output_images/Perspective_Transformation_Pipeline.png

[image6]: ./output_images/Sliding_Window_Lane_Detection.png

[image7]: ./output_images/Draw_Lane_On_Original_Image.png

[image8]: ./output_images/Sliding_Window_Histogram.png

[image9]: ./output_images/Draw_Radius_on_Original_Image.png

[image10]: ./output_images/ChessboardUndistortion.png

[image11]: ./output_images/Previous_Polyfit.png

[video1]: ./project_video_output.mp4


### Camera Calibration

The code for this step is contained in the first two code cells of the IPython notebook located in "./AdvancedLaneFinding.ipynb"  

I started by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection. OpenCV function `cv2.findChessboardCorners` was used to detect the chess corners which were used as image points. The result of the operation is shown below for all calibration images"

![alt_text][image1]

Note: The images that do not appear in the display are those where the specificied no. of corners were not detected.

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image10]


### Pipeline (single images)

#### Distortion Correction

An example of undistortion on a test image is shown below

![alt text][image4]

The image shows the effect of undistortion and it can be visualized by looking at the hood of the car

#### Color and gradient thresholding

I used a combination of color and gradient thresholds to generate a binary image (code cell 6).  Here's an example of my output for this step.

![alt text][image2]

The blue color in the stacked thresholds image shows edge detection using s-channnel thresholding by converting image into HLS domain. The green color in the stacked threshold image shows edge detection using l channel thresholding . The combination of s-channel threshold and l channel threshold does a resonable job for edge detection for lanes in image but there are some noise pixels in the image that can interfere with lane detection. To take care of some noise pixels, I used combination of Red color thresholding and s-channel thresholding. I also used sx graient thresholding for detecting white dashed lines

#### Perspective Transformation

The code for my perspective transform includes a function called `unwrap()`, which appears in code cell 8  in the Ipython notebook. The `unwrap()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

```python
h= img.shape[0]
w = img.shape[1]
src = np.float32([(190,720),(570,460),(710,460),(1180,720)])
dst = np.float32([(360,h),(360,0),(w-360,0),(w-360,h)])
```

This resulted in the following source and destination points:


| Source        | Destination   | 

|:-------------:|:-------------:| 

| 190, 720      | 360, 720      | 

| 575, 464      | 360, 0        |

| 707, 464      | 920, 0        |

| 1180, 720     | 920, 720      |


I verified that my perspective transform was working as expected by drawing the `src` points onto a test image and made sure that the lane lines appear parallel in the warped image.

![alt text][image5]


####  Lane Detection and Polynomial fit through left and right lane

I used `sliding_window`and `finding_fit_using_previous_lanefit` functions to find lane pixels and fit a 2nd order polynomial through the detected pixels.`sliding_window` fucntion first calculates the histogram at bottom half of image and then finds the peaks ofthe left and right halves of the hisotogram and those points will be considered as starting point so left and right lanes as shown below

![alt text][image8]

This histogram plot shows that left lane starts around 400 and right lane starts around 880 . This is found by summing the white pixels in a warped image  along image height for all the points on the image width. This function then selects ten sliding windows of particular height and width to identify pixels, each centered at the midpoint of previous window . After finding the left and right lane pixels, the second order polynomial is fitted through the pixels as shown below :

![alt text][image6]

In the image above, The sliding windows are shown in green, left and right lane  pixels are shown in red and blue respectively and yellow color shows curve fit through left and right lanes

The `finding_fit_using_previous_lanefit` function skips  the sliding window steps by using the information from previous frame and searching within  the vicnity of right and left lane fit detected in previous video frame . The image below shows the appication of this function. 

![alt_text][image11]

The green color in the above image shows the vicnity in which lane pixels were searched and fitted, red color shows the left lane pixels, blue color shows the right lane pixels and yellow color shows the fit through left and right lane pixels.

#### Calculation of the radius of curvature of the lane and the position of the vehicle with respect to center.

The radius of curvature is calculated in the code cell titled "Calculation of Radius of Curvature "

```python

left_curverad = ((1 + (2*left_fit_cr[0]*y_eval*ym_per_pix + left_fit_cr[1])**2)**1.5) / np.absolute(2*left_fit_cr[0])
right_curverad = ((1 + (2*right_fit_cr[0]*y_eval*ym_per_pix + right_fit_cr[1])**2)**1.5) / np.absolute(2*right_fit_cr[0])
rad_curv = (left_curverad+right_curverad)/2

```
In the code above, the radius of curvature is calulated as average of radius of curvature of left and right lane. 
left_fit_cr and right_fit_cr are coeffecients of second order polynomial fitted through detected lane pixels in an image. These pixel locations are first converted in meters before curve fitting using `np.polyfit`

The position of the vehicle with respect to the center of the lane is calculated in the code cell titled "Calculation of Vehicle Offset "

```python

# left and right lane intercept along image width 
left_lane_intercept = left_fit[0]*y_eval**2 + left_fit[1]*y_eval + left_fit[2]
right_lane_intercept = right_fit[0]*y_eval**2 + right_fit[1]*y_eval + right_fit[2]
# Lane midpoint in metres
lane_midpoint = ((left_lane_intercept + right_lane_intercept)/2)*xm_per_pix

# Calculating Lane Offset
Lane_Offset = Car_Centre - lane_midpoint

```
Vehicle offset with respect to centre is calculated as difference between car positon cente and lane cente. The lane centre is calculated as midpoint of left and right lane . These calcualtions are done in metres.

#### Example image of result plotted back down onto the road such that the lane area is identified clearly

This step is implemented in code cell titled "Drawing Lane on Undistorted image " and an example of an image with lane drawn is shown below

![alt text][image7]

After drawing lane, radius of curvature and vehicle offset is written on the top left of the image and the code for the same is provided in code cell titled "Function to draw lane as well as calculate & write rad of curvature & vehicle offset " . An example of result of using this funciton is shown below

![alt text][image9]

---


### Pipeline (video)

#### Final video output.  

Here's a [link to my video result](./project_video_output.mp4)

---

### Discussion

#### Potential Issues

The issues I faced was due to presence of different lightning conditions and shadows.It took some time to choose the right color and gradient threshold methods .I could further improve the video output by  further reducing noise in the output of color and gradient threshold technique, smoothening the video using the averageing or weightage technique and including more sanity check such as radius of curvature on the detected lanes.

