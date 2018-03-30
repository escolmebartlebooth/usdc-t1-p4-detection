**Advanced Lane Finding Project**

The goals / steps of this project were the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

[//]: # (Image References)

[image1]: ./output_images/camera_calibration.png "Calibration"
[image2]: ./output_images/distortion_correction.png "Distortion Correction"
[image3]: ./output_images/sobel_abs.png "Thresholds"
[image4]: ./output_images/sobel_magdir.png "Thresholds"
[image5]: ./output_images/color_select.png "Thresholds"
[image6]: ./output_images/combined_select.png "Thresholds"
[image7]: ./examples/warped_histogram.png "Warp Example"

[image5]: ./examples/color_fit_lines.jpg "Fit Visual"
[image6]: ./examples/example_output.jpg "Output"
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Section 1: Camera Calibration

The code for this step is contained in the first section the IPython notebook located in "usdc-advanced-lane-detection-final.iypnb".

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function. I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result:

![alt text][image1]

### Pipeline (single images)

The pipeline is a series of functions for undistorting, threshholding and transforming images so that lines can be detected. The code for each section are:
+ calibration - cell #3
+ undistortion - cell #4
+ sobel absolute threshold - cell #6
+ sobel magnitude threshold - cell #7
+ sobel direction threshold - cell #8
+ sobel hls color select - cell #9
+ perspective transform - cell #11
+ apply a histogram - cell #13

after this point, various test processing is done on all of the test image files.

#### 1. Provide an example of a distortion-corrected image.

Distortion correction takes calibration coefficients generated from the calibration function and applies them to the original image using open cv undistort function as shown below:
![alt text][image2]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

Thresholds are applied to images to isolate the facets of a lane line and to mask other elements of the image. This is made more difficult by varying road conditions such as shadows, different light conditions, different road lane colors and lengths, road condition (dirt, etc), and so on.

Each threshold takes an undistorted image and then performs different threshold paramterised techniques followed by then scaling the resulting pixels to (0,1) to provide a thresholded image.

In the image below, you can see the result of thresholding using:
+ sobel absolute in the x and y directions
+ sobel magnitude and direction thresholding
+ color space thresholding in the hls (s channel) and hsv (s channel) spaces

![alt text][image3]
![alt text][image4]
![alt text][image5]

The point about this part of the project was to find a set of parameters and combination of thresholds which would result in the best isolation of lane lines. This isolation could be best seen by applying a transform and taking the histogram of the transformed image across all test images to see which combination achieved the sharpest outline of the 2 lane lines of interest.

In code cell #15, combinations were tried and in code cells #16 thro' #20 a set of tuning experiments were carried out for each threshold and 1 combination (I believed that the best combination was sobel_abs_x and hls_color_select through experimentation not shown in the notebook)

The tuning parameters for each threshold were:
+ sobel_abs: min and max pixel values and kernel size - I found 20, 100 gave the best visual isolation for the X orientation. sobel_abs detects gradient intensity of grayscale images in the x and y planes. I didn't change kernel size.
+ sobel_mag: min and max pixel values and kernel size - I didn't find sobel mag as good at isolating the lines in a visual sense as sobel abs
+ sobel_dir: min and max pixel values - to the eye a very noisy image was produced but the lane lines were evident in the output image
+ hls and hsv color selection: I only experimented with the s channel of both color spaces and found the hls space slightly better at isolating broken lines than the hsv space

The combination of sobel_abs in the x direction with a threshold of (20,100) and hls color select with a combination of (180,255) was chosen as the candidate threshold:

![alt text][image6]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The approach to finding lanes and then fitting a line to the detected lanes is achieved by transforming the threshold image so that we get a bird's eye view of the road surface - zoomed onto the lane.

This is in cell #11. I chose to hard code the source and destination points. Using opencv functions it is possible to transform in both directions so that when lines are detected that can be played back into the original image.

```python
    img_size = (img.shape[1], img.shape[0])
    mid_x = img_size[0]//2
    top_width = 95
    bottom_width = 450
    top_start = 2*img_size[1]//3

    src_points = [
        (mid_x+top_width, top_start),
        (mid_x+bottom_width, img_size[1]),
        (mid_x-bottom_width, img_size[1]),
        (mid_x-top_width, top_start)
    ]

    dst_points = [
        (mid_x+bottom_width, 0),
        (mid_x+bottom_width, img_size[1]),
        (mid_x-bottom_width, img_size[1]),
        (mid_x-bottom_width, 0)
    ]
```

This resulted in the following source and destination points:

| Source        | Destination   |
|:-------------:|:-------------:|
| 585, 480      | 190, 0        |
| 190, 720      | 190, 720      |
| 1090, 720     | 1090, 720      |
| 775, 480      | 1090, 0        |

From the image below, the lines can be seen to be parallel in the transformed image and so the transform looks to be working.

![alt text][image7]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

...

![alt text][image8]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

...

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

...

![alt text][image9]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.
