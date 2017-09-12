##Advanced Lane Finding Project**

The goals / steps of this project are the following:

* Compute the camera calibration matrix and distortion coefficients given a set of chessboard images.
* Apply a distortion correction to raw images.
* Use color transforms, gradients, etc., to create a thresholded binary image.
* Apply a perspective transform to rectify binary image ("birds-eye view").
* Detect lane pixels and fit to find the lane boundary.
* Determine the curvature of the lane and vehicle position with respect to center.
* Warp the detected lane boundaries back onto the original image.
* Output visual display of the lane boundaries and numerical estimation of lane curvature and vehicle position.

**The code is in file advancedlanelines4.ipynb

[//]: # (Image References)

[image1]: ./output_images/camera_calibration.jpg
[image2]: ./output_images/straightline1.jpg
[image3]: ./output_images/test2.jpg
[video1]: ./project_output_colour_pipeline_v3_2.mp4

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the code cells 4 - 7 of the IPython notebook.

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
Target FIle: straightline1.jpg (first row of images)
![alt text][image2]

Target File: test2.jpg (first row of images)
![alt text][image3]

First we calibrate the camera and generate the camera matrix(mtx) and distortion matrix(dist)
Then we apply the mtx and dist to an image to get the undistorted image. 

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

I used thresholding as follows:
I found Gradient in x  and gradient direction. 
Used HSL color spaces focussing on s and l color space
Used R and G color spaces for white and yellow detection

The thresholding is in function apply_thresholds_v2.


I used a combination of color & gradient thresholds to generate a binary image Code is in Cell 8 .  The code is in functions: rg_threshold, hsl_threshold, abs_sobel_threshold, dir_threshold

Target FIle: straightline1.jpg (second row first images)
![alt text][image2]

Target File: test2.jpg (second row first images)
![alt text][image3]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

I use the following piece of code to do a perspective transform
```warped = cv2.warpPerspective(masked_image, placeme.M, (imshape[1], imshape[0]), flags=cv2.INTER_LINEAR)```

masked_image is generated as follows:
```masked_image = region_of_interest(rglsx_binary, vertices)```

The region_of_interest function is defined in 10th cell from top.
I have tried to use a slightly enhanced way that just hardcoding the src and dst points
I draw a equliateral parallelogram. Then I find the center of each side and rotate the parallelogram sides till it becomes rectangular. I find the x,y points corresponding to the dst by drawing a straight line to the top and bottom . In effect these become the dst points.
I have observed that doing a vanilla parrallelogram to rectangle tranformation may not effectively work for the perspective transform. So I allowed for a correction factor. In my case the correction factor is 40 px. 

The code is in functions : find_vertices, calculate_dst_for_perspective_transform, calculate_perspective_matrix and region_of_interest. It is in the 10th cell from top

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

Target FIle: straightline1.jpg (second row second image and third row first image)
![alt text][image2]

Target File: test2.jpg (second row second image and third row first image)
![alt text][image3]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?


To identify lane lines I used histogram of lower half and then divided it to 10 windows. For each window found the non zero pixels and appended it to list. Using these pixels, I fitted a 
second order polynomial. 

The code is in functions: findpeaks_v2, figure_out_line_predictions

Target FIle: straightline1.jpg (third row first image)
![alt text][image2]

Target File: test2.jpg (third row first image)
![alt text][image3]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

The code to measure curvature is in function: measure_radius_of_curvature

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in lines # through # in my code in `yet_another_file.py` in the function `map_lane()`.  Here is an example of my result on a test image:

Target FIle: straightline1.jpg (third row second image)
![alt text][image2]

Target File: test2.jpg (third row second image)
![alt text][image3]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

I have tried multiple iterations of pipeline code as evidenced by functions: pipeline_v2 and pipeline_v3. The differences are thresholding considerations, masking, etc

Function: pipeline_v3 was used to produce the desired output.

Here's a [link to my video result](./project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

Here I'll talk about the approach I took, what techniques I used, what worked and why, where the pipeline might fail and how I might improve it if I were going to pursue this project further. 

The issue I see is one of deciding on the src, dst points for perspective transform. Instead of manually choosing it, maybe we can run it through a CNN and find a good set or src/dst points

ANother thing is the usage of averaging across frames, averaging is pretty crude better method could be weighted average.

Here im using a combination of thresholding, a appropriate thresholding was found with trail-and-error , Im not mucha fan of trial and error. I was hoping to find some thechnique/s that will help offset this manual process. If given more time, this is one area where I would like to improve the pipeline