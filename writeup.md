

## Self Driving Car Engineer NanoDegree Program

---

​

**Advanced Lane Finding Project**

​

The goals / steps of this project are the following:

​

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.

* Apply a distortion correction to raw images.

* Use color transforms, gradients, etc., to create a thresholded binary image.

* Apply a perspective transform to rectify binary image ("birds-eye view").

* Detect lane pixels and fit to find the lane boundary.

* Determine the curvature of the lane and vehicle position with respect to center.

* Warp the detected lane boundaries back onto the original image.

* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

​

[//]: # (Image References)

​

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

​

​

### Camera Calibration

​

The code for this step is contained in the first two code cells of the IPython notebook located in "./AdvancedLaneFinding.ipynb"  

​

I started by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection. OpenCV function `cv2.findChessboardCorners` was used to detect the chess corners which were used as image points. The result of the operation is shown below for all calibration images"

![alt_text][image1]

​

Note: The images that do not appear in the display are those where the specificied no. of corners were not detected.

​

​

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

​

![alt text][image10]

​

### Pipeline (single images)

​

#### Distortion Correction

​

To demonstrate this step, I will describe how I applied the distortion correction to one of the test images like this one:

![alt text][image4]

​

The effect of undistortion is subtle, but can be perceived from the difference in shape of the hood of the car at the bottom corners of the image.

​

#### Color and gradient thresholding

​

I used a combination of color and gradient thresholds to generate a binary image (code cell 6).  Here's an example of my output for this step.

​

![alt text][image2]

​

The blue color in the stacked thresholds image shows edge detection using s-channnel thresholding by converting image into HLS domain. The green color in the stacked threshold image shows edge detection using l channel thresholding . The combination of s-channel threshold and l channel threshold does a resonable job for edge detection for lanes in image but there are some noise pixels in the image that can interfere with lane detection. To take care of some noise pixels, I used combination of Red color thresholding and s-channel thresholding.

​
#### Perspective Transformation

​

The code for my perspective transform includes a function called `unwrap()`, which appears in code cell in the Ipython notebook. The `unwrap()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

​

```python

​

src = np.float32([(190,720),(575,464),(707,464),(1180,720)])
dst = np.float32([(400,h),
                  (400,0),
                  (w-400,0),
                  (w-400,h)])

```

​

This resulted in the following source and destination points:

​

| Source        | Destination   | 

|:-------------:|:-------------:| 

| 190, 720      | 400, 720      | 

| 575, 464      | 400, 0        |

| 707, 464      | 880, 0        |

| 1180, 720     | 880, 720      |

​

I verified that my perspective transform was working as expected by drawing the `src` points onto a test image and made sure that the lane lines appear parallel in the warped image.

​

![alt text][image5]

​

####  Lane Detection and Polynomial fit through left and right lane

​

The functions `sliding_window`and `polyfit_using_prev_fit` , which identify lane lines and fit a second order polynomial to both right and left lane lines, are clearly labeled in the Jupyter notebook as "Sliding Window Polyfit" and "Polyfit Using Fit from Previous Frame". The first of these computes a histogram of the bottom half of the image and finds the bottom-most x position (or "base") of the left and right lane lines. Originally these locations were identified from the local maxima of the left and right halves of the histogram, but in my final implementation I changed these to quarters of the histogram just left and right of the midpoint. This helped to reject lines from adjacent lanes. The function then identifies ten windows from which to identify lane pixels, each one centered on the midpoint of the pixels from the window below. This effectively "follows" the lane lines up to the top of the binary image, and speeds processing by only searching for activated pixels over a small portion of the image. Pixels belonging to each lane line are identified and the Numpy polyfit() method fits a second order polynomial to each set of pixels. The image below demonstrates how this process works:

​

​

![alt text][image6]

​

The image below depicts the histogram generated by sliding_window; the resulting base points for the left and right lanes - the two peaks nearest the center - are clearly visible:

​

![alt_text][image8]

​

The `polyfit_using_prev_fit`  performs basically the same task, but alleviates much difficulty of the search process by leveraging a previous fit (from a previous video frame, for example) and only searching for lane pixels within a certain range of that fit. The image below demonstrates this - the green shaded area is the range from the previous fit, and the yellow lines and red and blue pixels are from the current image:

​

![alt_text][image11]

​

​

#### Calculation of the radius of curvature of the lane and the position of the vehicle with respect to center.

​

The radius of curvature is calculated in the code cell titled "Radius of Curvature and Offset from Lane Center  using this line of code (altered for clarity):

​

```python

curve_radius = ((1 + (2*fit[0]*y_0*y_meters_per_pixel + fit[1])**2)**1.5) / np.absolute(2*fit[0])

```

In this example, fit[0] is the first coefficient (the y-squared coefficient) of the second order polynomial fit, and fit[1] is the second (y) coefficient. y_0 is the y position within the image upon which the curvature calculation is based (the bottom-most y - the position of the car in the image - was chosen). y_meters_per_pixel is the factor used for converting from pixels to meters. This conversion was also used to generate a new fit with coefficients in terms of meters.

​

The position of the vehicle with respect to the center of the lane is calculated with the following lines of code:

```python

lane_center_position = (r_fit_x_int + l_fit_x_int) /2

center_dist = (car_position - lane_center_position) * x_meters_per_pix

```

r_fit_x_int and l_fit_x_int are the x-intercepts of the right and left fits, respectively. This requires evaluating the fit at the maximum y value (719, in this case - the bottom of the image) because the minimum y value is actually at the top (otherwise, the constant coefficient of each fit would have sufficed). The car position is the difference between these intercept points and the image midpoint (assuming that the camera is mounted at the center of the vehicle).

​

#### Example image of result plotted back down onto the road such that the lane area is identified clearly

​

I implemented this step in the code cells titled "Draw Lane On Image" and "Draw Radius of Curvature on Image" in the Jupyter notebook. A polygon is generated based on plots of the left and right fits, warped back to the perspective of the original image using the inverse perspective matrix Minv and overlaid onto the original image. The image below is an example of the results of the draw_lane function:

![alt text][image7]

​

Below is an example of the results of the `draw_data` function, which writes text identifying the curvature radius and vehicle position data onto the original image:

![alt text][image9]

---

​

### Pipeline (video)

​

#### Final video output.  

​

Here's a [link to my video result](./project_video_output.mp4)

​

---

​

### Discussion

​

#### Potential Issues

​

The problems I encountered were almost exclusively due to lighting conditions, shadows, discoloration, etc.I used fixed threshold parameters to obtain binary images.THe other issue I encountered was smoothening the video output. I used previous fits to smoothen the video. Perphaps I can use other ways of detecting false lane detections and use them to make my algorithm more robust.


