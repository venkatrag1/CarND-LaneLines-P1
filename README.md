# **Finding Lane Lines on the Road**

---

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[image1]: ./writeup_ref_images/img1-colorSelect.jpg "Color Selection"
[image2]: ./writeup_ref_images/img2-blurGray.jpg "Blur Grayscale"
[image3]: ./writeup_ref_images/img3-canny.jpg "Canny Edge"
[image4]: ./writeup_ref_images/img4-ROI-hough.jpg "ROI mask and Hough transform"
[image5]: ./writeup_ref_images/img5-final.jpg "Final Image"

---

### Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

My pipeline consisted of the following steps.

1. Apply a color selection mask to original image with high thresholds for red and green, and relatively low threshold for blue. This step was actually added at the very end to avoid detecting dark lines for the Challenge video. Since the lane lines we are trying to detect are primarily yellow and white, by filtering out objects with lower RG thresholds, we significantly reduce the scope of future steps such as canny edge detection.
![alt text][image1]

2. Convert the image to grayscale and apply GaussianBlur. A high value of 7 was chosen for the Kernel size as blurring over these many pixels helped the canny edge detector ignore the thin white horizontal chalk lines for the solidYellowLeft video.
![alt text][image2]

3. Now apply canny edge detector to the grayscale image from previous step. Here we arrived at a high threshold of 180 by sweeping, to avoid detecting lower gradient lines on the shoulder of the road.
![alt text][image3]

4. Apply region of interest (ROI) mask to the canny edges image. The ROI mask is chosen based on position of camera in the car and height to which road can be seen into distance. The ROI mask starts at bottom of the image and goes to upto 40 pixels below the middle of image height, assuming a trapezoidal shape. The region of interest is shown using purple lines in the image below. After blackening out pixels outside the ROI, the edges image is piped through Hough transform. Hough transform is invoked with 1 pixed and 1 degree as rho and theta resolutions, for high accuracy. Minimum line length was determined to be 20 through sweeping, to avoid detecting car as lane line.
![alt text][image4]

5. By default, the draw_lines method overlays multiple lines for each detected edge onto the original image using color red and thickness 2.
In order to draw a single line on the left and right lanes, I modified the draw_lines() function by computing the average for the slope and intercept for all the detected left lines and right lines. Lines less than 15 degrees on either side are ignored since this will be true only if we're driving perpendicular to the direction of traffic, which then becomes an error handling case. This filtering helps performance on the Challenge video. A margin equal to overlaid line thickness is chosen for all edges. The bottom y is always chosen to be bottom of image, and the top y is chosen to be the max value seen on either left or right side, for sake of symmetry, and a line is drawn connecting the y points on either side using the average slope and intercept.
![alt text][image5]

### 2. Identify potential shortcomings with your current pipeline


A primary shortcoming of this pipeline stems from the fact that this uses a pure image processing approach instead of any adaptive tuning of parameters. This would make the pipeline more suited for post processing of videos and unsuitable for learning in new environments since the parameters for detection are hand tuned through experimentation here.

For instance, different lighting conditions such as in the challenge video can cause the pipeline to fail.

The pipeline also used a crude approach to deal with noisy lines by setting a high value for minimum line width for hough transform and through aggressive masking.

Finally, this pipeline also detects lines, and so it is not very useful in roads with turns as it can't ahead well in those cases, causing the car to slow down significantly on edges.


### 3. Suggest possible improvements to your pipeline

A possible improvement would be to better handle outliers in terms of edges detected due to miscellaneous white lines by using a better statistical approach than averaging. By increasing storage for all lines detected by canny on a frame, we could potentially use median to get the middle slope and corresponding intercept instead of averaging.

This would also allow us to go to a lower value for minimum line length allowing the lane detection to fit better.

Another potential improvement would be to handle changes in lighting conditions better by a smarter masking approach. 
