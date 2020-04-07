## Project - Advanced Lane Finding Writeup
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

[image1]: ./output_images/Cam_Cal-Orig.jpg "Camera Calibration Original"
[image2]: ./ouptut_images/Cam_Cal-Warp.jpg "Camera Calibration Warped"
[image3]: ./output_images/Undist-Orig.jpg "Undistorted Original"
[image4]: ./ouptut_images/Undist-New.jpg "Undistorted New"
[image5]: ./output_images/Binary-Orig.jpg "Binary Original"
[image6]: ./ouptut_images/Binary-Warp.jpg "Binary Warped"
[image7]: ./output_images/Warped-Orig.jpg "Warped Original"
[image8]: ./ouptut_images/Warped-Warp.jpg "Warped New"
[image9]: ./output_images/Poly-Orig.jpg "Polynomial Original"
[image10]: ./ouptut_images/Poly-Warp.jpg "Polynomial Warped"
[image11]: ./output_images/Final-Orig.jpg "Final Original"
[image12]: ./ouptut_images/Final-Warp.jpg "Final New"
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

he code for this step is contained in cells 4 - 6 of the IPython notebook located in "MDP - Advanced Lane Line Project".  

I start by defining a function to calibrate the camera, cam_cal.  Within this function, I bring in the entire directory of camera calibration images, for each image, I convert to gray scale, find the chess board corners and build the object points and image points arrays.  I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

Cell 6 will perform the actual calibration.
Cell 7 will demonstrate unwarping the image using the matrix that was determined in Cell 6.

![alt text][image1][image2]

### Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

This step was very straightforward.  Once the camera was calibrated, we were able to use the following code:
    undist = cv2.undistort(image, mtx, dist, None, mtx)
To produce the following images:

![alt text][image3][image4]

#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

Now creating the binary image was interesting.  I tried to design a method that would allow me to easily change my parameters to find the best combination.  So I created 4 functions
	abs_sobel_thresh
	mag_threshold
	dir_threshold
	hls_select
Using different combination of each of these functions, I was able to narrow down the best (current) combination and value set.  Each function has it own set of parameters, so I created a wrapper function:
	binary_image
This function takes arguments to turn each different function on or off, along with passing the needed parameters into each.  Finally, it layers each active image into a single image.  
Ultimately, I chose a combination Sobel(X orientation) + HLS(S channel) + magnitude to get my binary image.  Within the process_image function, all of my variables choices are shown.

![alt text][image5][image6]

#### 3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called warp_image().  The function takes as inputs an image (`img`), as well as points for the trapezoid, the matrix and distortion coefficients as well.  The function then returns the warped image and inverse M matrix.  My method for building the trapeziod was based on percentages of the image size. So the points fed into the function are the "top height" in %,  width 1 in % and width 2 in %.  Then I would use these factors to determine how high, and wide the different lengths of the trapezoid were.  This method would always work from the center of the image and use the factors to build using that.

![alt text][image7][image8]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

I created a function called fit_polynomial to handle the line creation based on the points.  This function took quite a few inputs and also returned quite a few outputs.  The reason for this was to be able to send in the previous polynomials, to allows for improved processing and smoother lines.  Inside of this function, there are 2 different function calls (one for lane-line-pixels & one for when you already have a previously fit polynomial). 

In addition, I later added a feature to force lane-line-pixel method after X # of frames, to insure that the polynomial wasn't drifting too far and causing inconsistent results.

![alt text][image9][image10]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

I created function measure_curvature_real to handle the radius of curvature.  In the fit_polynomial, I had already calulated the pixels per meter (for x & y), so those are sent into the function as inputs, along with the polynomial that was recalculated for meters in the same function.  From there, it just plug everything into the equation to return the left and right radia.

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step in the fit_polynomial function as well.  Since all the data was already available, this function returns the final transformed image, as well as the poly fit "top-down" view:

![alt text][image11][image12]

---

### Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).

Here's a [link to my video result](./project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The biggest challenge for me with this project was working through the binary image filters.  There were to many possible options to try, that even now after spending 40+ hours on this project, I'm still sure I could tune these better.  
My pipeline certainly fails in the challenge video.  The example where there is a concrete median that is close enough to the lane lines looks like it could be a lane line, which would clearly be an issue.  In addition, road construction with "roughly" straight lines of tar on the road also appear as a lane.

The next things I would try would be 
1.  to isolate the slopes of all possible line, and find 2 lines with approximately the same slope.  
2.  I would also use the pixels to meters conversion to find the 2 lines closest to the lane width.  
3.  I would also attempt to isolate by the HUE of white or yellow as well.  
4.  I would experiment with a dynamic trapezoid shape that could evolve based on previous images to help the lanes smooth over.  

Short story, there are a lot of additional pieces I would add, if I had the time.