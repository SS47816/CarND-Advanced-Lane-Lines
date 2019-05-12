## Writeup

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

[image1]: ./output_images/undistort_output.jpg "Undistort"
[image2]: ./output_images/undistorted.jpg "Undistorted"
[image3]: ./output_images/warped.jpg "Road Transformed"
[image4]: ./output_images/combined_threshed.jpg "Binary Example"
[image5]: ./output_images/line_fit_result.jpg "Fit Visual"
[image6]: ./output_images/output_image.jpg "Output"
[video1]: ./output_videos/project_video.mp4 "Project Video"
[video2]: ./output_videos/challenge_video.mp4 "Challenge Video"
[video3]: ./output_videos/harder_challenge_video.mp4 "Harder Challenge Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---

### Writeup / README

#### 1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

You're reading it!

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the IPython notebook located in "./P2.ipynb", **Step 1**.  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the calibration image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

The code for this step is contained in the IPython notebook located in "./P2.ipynb", **Step 1**.  

In this step, I defined a function called `cal_undistort()` to apply undistortion to the raw image. This function takes in the `img`, which is the raw testing image, and the `objpoints` `imgpoints` given in step 1. The function uses `cv2.calibrateCamera()` to calculate distortion coefficients, and `cv2.undistort()` to undistort the raw image. 

Here is an example of the undistorted image 
![alt text][image2]



#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

The code for this step is contained in the IPython notebook located in "./P2.ipynb", **Step 2**

I used a combination of color and gradient thresholds to generate a binary image. 

First, in the function `combined_threshold()`, two color channels, R and S, were seletced for their good complementary performance in various senarios. Each of these two channels then went through magnitude & direction thresholding via `gradients_threshold()` function to generate a binary image, where logic used is "AND". Then, the two binary images from each channel are combined together with the "OR" logical operator, to make the detection algorithm more addaptive to different senarios. 

.  Here's an example of my output for this step.  (note: this is not actually from one of the test images)

![alt text][image3]


#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for this step is contained in the IPython notebook located in "./P2.ipynb", **Step 3**.

The code for my perspective transform includes a function called `perspective_transform()`, which takes an image `img`, transformation matrix `M`, and the `imshape`, which is the shape of the raw image (in (x,y) order) as inputs. It uses `cv2.warpPerspective()` to warp the image to a top-down view. 

The second funtion in this step is called `calculate_transform_matrix()`, which takes in `imshape`(the shape of the raw image (in (x,y) order), and six other inputs, which are used to define four trapezium-shape `src` points & four rectangle-shape `dst` points. The six inputs are `left`, `right`(x coordinates of the bottom left/right `src` point), `top`, `bot`(y coordinates of all the top/bottom `src` point), `top_left`(x coordinate of the top left `src` point), `new_left`(x coordinate of the top left & bottom left `dst` points). The y coordinates of top left & top right `dst` points by default are 0, while that of the bottom left & bottom right points are `imshape[1]`. The code to define the points is shown below: 

```python

src_left_top = (top_left, top)
src_right_top = (imshape[0] - top_left, top)
src_right_bot = (right, bot)
src_left_bot = (left, bot)
src = np.float32([[src_left_top, src_right_top, src_right_bot, src_left_bot]])
new_right = imshape[0] - new_left
dst_left_top = (new_left, 0)
dst_right_top = (new_right, 0)
dst_right_bot = (new_right, imshape[1])
dst_left_bot = (new_left, imshape[1])
dst = np.float32([[dst_left_top, dst_right_top, dst_right_bot, dst_left_bot]])

```

This resulted in the following source and destination points:

| Source        | Destination   | 
|:-------------:|:-------------:| 
| 256, 684      | 200, 720      | 
| 570, 468      | 200, 0        |
| 710, 468      | 1080, 0       |
| 1024, 684     | 1080, 720     |

This function returns the transformation matrix `M`, and the inverse transformation matrix `Minv`.

Here is an example of the warped lane image
![alt text][image4]

In addition, to facilitate the visualization of the drivable area, a function called `calculate_center_margin` was created. It takes in the shape of the image (`imshape`), the x coordinate of the bottom left `dst` point(`new_left`), and calculate the "half of the lane width in pixels". 

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

The code for this step is contained in the IPython notebook located in "./P2.ipynb", **Step 4**

In this step, I implemented both the "sliding window" method and the "convolution window" method. 

In 4.a) The function `find_lane_pixels()` first creates a histogram for the bottom quarter of the warped binary image `binary_warped`, find the left and right peaks and uses them as starting points for left and right windows. The windows at each level sliding left and right within the `margin` and stop when the number of pixels reaches `minpix`. Then, the function retruns the x,y coordinates of the left and right window centroids(`leftx`, `lefty`, `rightx`, `righty`), together with the ploted image(`out_img`) to `fit_polynomial()` to generate the left and right fit lane points and two polynomials describing them(`left_fit`, `right_fit`). I used the average of these left and right fit lane points to simulate some center fit lane points and a polynomial called `center_fit`. Finally, the function draws the left and right polynomials, and the center polynomial, with a thickness equals to `2*center_margin`, and returns the final output image `out_img` and the three polynomials. 

Here is an example output image from this function:

![alt text][image5]

In 4.b) The function `convolution_window()` implements the "convolution window" method, with the help of the other two functions: `window_mask()`, and `find_window_centroids()`. Genreal, this method is simmilar the "sliding window" method described in 4.a). Only in this time, the criteria for sliding windows changed to maximising the number of hot pixels in the window, instead of only reaching a fixed threshold value. 

In 4.c) Once the lane lines have been found, the function `search_around_poly()` will take over, to continue searching for lane lines around the previous results, with the help of function `fit_poly()`.

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The code for this step is contained in the IPython notebook located in "./P2.ipynb", **Step 5**

Function`calculate_distance2pixel()` was creataed to transform the relationship between the warped image pixels and the real distance they represent in real world. While `measure_curvature_real()` is the main function that takes in the shape of the image(`imshape`), the `center_margin`, and the polynomial that describes the center lane(`cneter_fit`). The output of this function are the radius of the lane curveture(`center_curverad`), the absolute distance bwtween the camera center and the lane center(`abs_track_error`), and whether the vehicle is on the left or right side of the road(`position_flag`). 

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

The code for this step is contained in the IPython notebook located in "./P2.ipynb", **Step 6**

With the help of the function `perspective_transform()` in step 3, function `display_lane()` in this step takes in the undistored image(`raw_img`), the warped lane detection result image from step 4(`result`), the inverse transformation matrix `Minv`, the shape of the image `imshape`, and transform the warped lane image to the original view. Then, it uses `cv2.addWeighted()` function to combine stack the lane image and the undistorted original image.

The code for this step is contained in the IPython notebook located in "./P2.ipynb", **Step 7**

Function `add_text()` takes in the image from the previous step(`img`), `center_curverad`,`abs_track_error`, and `position_flag`, and display these values onto the `img`. 

Here is an example of my result on a test image:

![alt text][image6]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./output_videos/project_video.mp4)

[video1]: ./output_videos/project_video.mp4 "Project Video"
[video2]: ./output_videos/challenge_video.mp4 "Challenge Video"
[video3]: ./output_videos/harder_challenge_video.mp4 "Harder Challenge Video"

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further.  

**Known Issues**

This algorithm is likely to fail when

1. Failure due to the sliding window/convolutional window
    
    a. The lane lines are not continuous
    
    b. The curveture of the lane is too big
    
    c. The curveture of the lane go out of FOV
    
    d. Driving uphill/downhill
    
    
2. Failure due to the color thresholding
    
    a. The road color changes/shadow
    
    b. There are similar patterns besides the lane lines (barriers, walls, longitudinal gaps)
    
    c. Under extreme low/high light condition


**Possible Solutions**

1. Adjust the window settings/perspective transform settings
    
    a. The lane lines are not continuous
        i) Increase the height of the window 
        ii) When no/not enough hot pixels found in one window, continue the trend of the previous windows.
        
    b. The curveture of the lane is too big 
        i) Decrease the height of the window
        ii) Increase the width of the window
        iii) Rotatable window that can adapt the lane direction maybe?
        
    c. The curveture of the lane go out of FOV 
        i) Increase the camera FOV 
        ii) When no/not enough hot pixels found in one window, continue the trend of the previous windows.
        
    d. Driving uphill/downhill 
        i) Dynamic warpping aera
        ii) Dynamic region of interest
        iii) Prediction of the lane direction behind objects/obstacles

2. Adjust the color/gradient thresholding
    
    a. The road color changes/shadow 
        i) Color filter
        ii) Tune the S channel
        
    b. There are similar patterns besides the lane lines (barriers, walls, longitudinal gaps) 
        i) Color filter
        ii) Check if the left and right lane lines are parallel
        
    c. Under extreme low/high light condition 
        i) Dynamically adjust the lightness
        ii) Tune the S channel or try other color channels
        


