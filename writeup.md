#**Finding Lane Lines on the Road** 

This project was to create a pipeline to process an image and identify the left and right lane markings.

[image1]: ./examples/grayscale.png "Grayscale"
[image2]: ./examples/smoothed.png "Smoothed"
[image3]: ./examples/canny.png "Canny Edges"
[image4]: ./examples/roi_gray.png "ROI"
[image5]: ./examples/masked_edges.png "Masked Edges"
[image6]: ./examples/lines.png "Hough Lines"
[image7]: ./examples/computed_lines.png "Computed Lines"
[image8]: ./examples/final.png "Final Image"

##Pipeline Description

My pipeline consisted of 6 stages:

1. Converts the image to grayscale so as not to be dependent on color.
![alt text][image1]
2. Gaussian blur to smoothe the image, kernel size = 5
![alt text][image2]
3. Canny edge dectection to generate image of just edges. It is using Otsu's method for calculating threshold which seems to work better for the changing background intensity and lighting conditions of the "optional challange" video.
![alt text][image3]
4. Define a trapezoidal region of interest (ROI) in the area we would expect to find lane markings.  Hyperparameters defined as a percentage of the image to adjust to varying resolution input. Remove edges found in step 2 that are outside the ROI.
![alt text][image4] ![alt text][image5] 
5. Use Probabilistic Hough Transform to find lines in the image which should represent the edges of the lane markings.
![alt text][image6]
6. Using the Hough Lines from stage 5, find just 2 lines representing the left and right lane markings.
![alt text][image7] ![alt text][image8]

In order to draw a single line on the left and right lanes in stage 6, I do the following steps:

1. Loop over the lines and separate them into left and right lines based on whether the slope of the line is positive or negative, ignore any near-horizontal lines (slopes of magnitude less than .3)
2. For both the left and right lines, find a weighted average of the slopes and intercepts, using the line length as the weight.  The idea is that "long edges" are more representative of the lane markings than "short edges" which are more likely to be false positives.
3. The average slope/intercept for the left and right line found in step 2 become the "expected" left/right line.  Step 2 is repeated but this time ignoring lines that have a slope too far from the "expected".  This gets rid of more anomolous lines and makes for smoother results between frames.


##Potential shortcomings with the pipeline

Some short comings of the pipeline are:

* The left/right separatinge logic assumes left and right lines have different sign (positive/negative) slopes. This may not be the case while the car is in a tight curve
* The "weighted average" logic in stage 6 can cause a long, false positive edge to dominate the average.
* The pipeline assumes straight lines fit the lane markings, which may not be the case especially on a curving road.

##Possible improvements to the pipeline

Some possible improvements to the pipeline could be:

* Partition the image into a grid, assigning each line to the grid box they fall into based on center point.  The rows of the grid can be used to calculate different average slopes at different heights of the image to account for curves.   One should also be able to use the columns of the grid to separate lines into left and right without depending on slope.
* Plot the center points of the lines and then do a linear regression to find a "best fit" line.  This would allow horizontal lines (shuch as the horizonal edges of the dashed lane markers) to be included in the calculation.   Using this method can also be expanded to fit a curve ("spline") for curved roads.
* Use information from previous frames in the calculation of lines in the current frame.   The slope/intercept of the left/right lane markers should not change too much from frame to frame, so we should be able to use the line information from the previous frame in the current frame (e.g as part of a weighted average where the previous frame lines get a lower weight than the current frame).


