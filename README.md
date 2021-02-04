## Advanced Lane Finding
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

[image1]: ./examples/undistort_output.png "Undistorted"
[image2]: ./test_images/test1.jpg "Road Transformed"
[image3]: ./examples/binary_combo_example.jpg "Binary Example"
[image4]: ./examples/warped_straight_lines_check.jpg "Warp Example"
[image5]: ./examples/color_fit_lines.jpg "Fit Visual"
[image6]: ./examples/example_output_2.jpg "Output"

[image7]: ./examples/original_image.jpg "Original Image"
[image8]: ./examples/undistorted_image.jpg "Undistorted Image Output"
[image9]: ./examples/sobel_and_hls_thresholding_output.jpg "Sobel and Color Thresholding Output"
[image10]: ./examples/sobel_and_hls_thresholding_binary_output.jpg "Sobel and Color Thresholding Binary Output"
[image11]: ./examples/masked_sobel_and_hls_thresholding_binary_output.jpg "Masked Sobel and Color Thresholding Output"

[video1]: ./video_outputs/project_video.mp4 "Video"

### Camera Calibration

The code for this step is contained in the camera_calibration() function within the Jupyter notebook located in `./P2.ipynb`.

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)
The main pipeline for processing single images is contained within `process_image()` function and consists of the following steps:

#### 1. Undistorting the image
Using the distortion matrix obtained from the camera calibration, I used OpenCV function `cv2.undistort()` to undistort the image. A sample results is shown below:
![alt text][image7]
![alt text][image8]



#### 2. Applying color and gradient transforms and masking

I used a combination of color and gradient thresholds to generate a binary image. The thresholding is contained within the `sobel_hls_threshold()` function that takes the undistorted image from previous step as input. The color thresholding is defined on the HLS color maps and thresholds the H and S channels, whereas the gradient threshold acts on the x-direction. Here's an example of my output for this step:

![alt text][image9]
![alt text][image10]

Additionally I define a polygon mask using `vertex()` function to obtain the pixles only within a region of interest using `region_of_interest()` function. Result would look like this:
  
![alt text][image11]


#### 3. Perspective transform

Matrices of the perspective transform are obtained from `warp_parameters()` function and later used within `cv2.warpPerspective()` to get a top view of the road.

 I chose the hardcoded the source and destination points in the following manner:

```python
src = [[(img_size[0] // 6) - 3, img_size[1]], 
      [(img_size[0] // 2) - 43, (img_size[1] // 2) + 90], 
      [(img_size[0] // 2) + 43, (img_size[1] // 2) + 90], 
      [(img_size[0] * 5 // 6) + 34, img_size[1]]]
    
dst = [[(img_size[0] / 4), 0],
    [(img_size[0] / 4), img_size[1]],
    [(img_size[0] * 3 / 4), img_size[1]],
    [(img_size[0] * 3 / 4), 0]]
```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 210, 720      | 320, 720      | 
| 597, 450      | 320, 0        |
| 683, 450      | 960, 0        |
| 1100, 720     | 960, 720      |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image. To do this correctness check and to make the following plots just call `check_corners_unwarp()`

![alt text][image4]

#### 4. Finding lane-line pixels and fit their positions with a polynomial
As the next step lane-line pixels for each lane was found using the function `find_lane_pixels()` which takes the binary warped image of the previous step as input. These pixels are then added via the `add()` method to `left_lane` and `right_lane` which are instanes of the `Line` class. One of the tasks `add()` method does is it inherently implements a polynomial fit to these pixels as well and stores the coefficients A, B and C as shown in the image below.

![alt text][image5]

#### 5. Finding radius of curvature of the lane and the position of the vehicle with respect to center

When using the `add()` method in `Line` class to add pixels, in addition to polynomial fittind, the method also finds the curvature of each lane using `measure_curvature_meter()` function. 
Additionally, the position of the vehicle with respect to center lane is also shown as asn overlay on top of the original image. This distance is found within `road_overlay()` function.


#### 6. Overlay on the original image and the final results of the pipeline
In the final step `road_overlay()` draws the results on the original image as well overlaying the road curvature and lane center offset. Results for a single reference image is shown below: 


![alt text][image6]

---

### Pipeline (video)

#### 1. Link to the final video output using the pipeline explained above 

Here's a [link to my video result](./video_outputs/project_video.mp4)

---

