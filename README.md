[//]: # (Image References)

[image1]: ./output_images/undistort.png "Undistorted"
[image2]: ./output_images/test4.png "Road Transformed"
[image3]: ./output_images/binary_threshold.png "Binary Masks"
[image4]: ./output_images/warped.png "Warp Example"
[image5]: ./output_images/sliding_windows.png "Sliding Window Fit"
[image6]: ./output_images/line_margin.png "Line Margin Fit"
[image7]: ./output_images/drawn_lane.png "Drawn Lane"
[image8]: ./output_images/result.png "Output"

# Advanced Lane Finding
[![Udacity - Self-Driving Car NanoDegree](https://s3.amazonaws.com/udacity-sdc/github/shield-carnd.svg)](http://www.udacity.com/drive)

In this project, your goal is to write a software pipeline to identify the lane boundaries in a video

## Writeup / README

***Provide a Writeup / README that includes all the rubric points and how you addressed each one.***

You're reading it!

### Camera Calibration

***Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.***

The code for this step is contained in the code cell #2 of the IPython notebook "advanced_lane_lines.ipynb".  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result:

![alt text][image1]

### Pipeline (single images)

***1. Provide an example of a distortion-corrected image.***
I prepared a wrapper function around the `cv2.undistort()` so that I didn't have to send all the parameters for every image, now it can be called like `undistort(img)` and the result is something like the image below
![alt text][image2]

***2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image. Provide an example of a binary image result.***

I used a combination of color and gradient thresholds to generate a binary image (thresholding functions at cell #5 in the notebook).

In the cell #6 I test the values and visualize the result from the different gradient threshold

For the color threshold I apply a mask on the R channel of the RGB image and another mask on the L channel of the HLS image, then I combine them to get a final color mask, this is tested in cell #7

Here you can see the different masks and the final result.
![alt text][image3]

***3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.***

The code for my perspective transform includes a function called `warp()`, which appears in the code cell #9. The `warp()` function takes as inputs an image (`img`) and a Boolean value that lets me warp or unwarp the image. I choose to hardcode the source and destination points in the following manner:

| Source        | Destination   |
|:-------------:|:-------------:|
| 700, 460      | 980, 0        |
| 1000, 680     | 950, 720      |
| 300, 680      | 350, 720      |
| 580, 460      | 320, 0        |

I verified that my perspective transform was working as expected by drawing the `src` points onto a test image and its warped counterpart with vertical lines to verify that the lines appear parallel in the warped image.

![alt text][image4]

***4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?***

To detect the line I made 3 functions in cell #11. First I call `detect_lines()` that will check if the previous line was detected or not and use the correct method for finding the lane.

If there is no previous line it will use the sliding windows approach to look for the line pixels.

It works by first using an histogram of the bottom half of the image to locate the place with most points and setting the initial search window to this points.

Then the code keeps adding windows and looking for points, if more than 50 points are found the window gets relocated to the new center.

After all points have been identified a polynomial line its fitted across this points to get each of the lines.

This process can be visualized in the following image
![alt text][image5]

If the previous line was detected the process is much simpler and now the code only looks for points that are near the previous line fit.
![alt text][image6]

***5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.***

I did this in cell # 13 with the function `get_measures()`, there I get the points for the current line, then I convert them from pixels to meters and get the new coefficients.

With the new coefficients we can apply the formula for radius of curvature

For the position of the vehicle first I find the center of the lane and then compare it with the center of the image, if this is negative number then the car is to the left of the center.

***6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.***

In cell #14 I define the function `draw_lane()` where I draw the lane along with the 2 lines that where detected  

![alt text][image7]

Then in the main pipeline this gets unwarped and added to the original image
![alt text][image8]

***7. Sanity Check.***

In cell #15  I added some checks to decide if the line seems to be correct, I check for separation, the radius of curvature and the slope

***8. Smoothing.***

In cell #16 I define the function `smooth()` when it detects a line it will calculate the average coefficient and store it in the line.best_fit

---

### Pipeline (video)
My final pipeline is defined in cell #17,
it has the following steps
1. Undistort Image
2. Apply gradient and color threshold
3. Warp the image
4. Detect Lines
5. Calculate curve radius and distance from center
6. Check if line is valid
7. Smooth the lines
8. Draw the detected lane
9. Unwarp image
10. Print info on image

***Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).***

Here's a [link to my video result](https://www.youtube.com/watch?v=BcJRhQuBiQ4)

---

### Discussion

***Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?***

The code works well on the first video, it still has some difficulties with the shadows and different lighting conditions. this could be improved with further preprocessing the image and exploring some other color spaces

Points father away in the image are also difficult to detect maybe reducing the search area gives better results reducing the noise and this will also help with sharper turns

Sharp turns are also difficult as the code searches for points in a mostly vertical way, as mentioned earlier this may be solved by reducing the search area
