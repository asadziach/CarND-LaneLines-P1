# **Finding Lane Lines on the Road** 

## Asad Zia

---

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[orginal]: ./writeup_images/solidYellowCurve2.jpg
[yellow]: ./writeup_images/yellow.png
[white]: ./writeup_images/white.png
[yellow_white]: ./writeup_images/yellow_white.png
[canny]: ./writeup_images/edge_detection.png
[mask]: ./writeup_images/mask.png
[hough]: ./writeup_images/hough_lines.png
[output]: ./writeup_images/output.png

---

### Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

My pipeline consisted of following steps. 

1. Convert to HSV color space 
2. Select White and Yellow areas only by cv2.inRange
3. Apply Gaussian Blur
4. Detect Edges by Canny's method
5. Define Area of Interest and mask off everything outside
6. Run Hough Lines detection
7. Composite lane lines on top of original image

I started by having first part of my pipeline to convert the image in grayscale and then run Edge detection on it. It has having difficulty finding yellow lines so I switched to color selection method.
In order to draw a single line on the left and right lanes, I modified the draw_lines() function by first iterating through all the lines detected by Hough algorithm and for each line:

* Calculate slope by (y2-y1)/(x2-x1)
* Find center by (x2+x1)/2, (y2+y1)/2
* Drop the line if slope is less than 25&deg; and greater than 85&deg;
* If slope is positive then add to "right_slopes" list, add corresponding center to "right_centers" list
* If slope is negative then add to "left_slopes" list, add corresponding center to "left_centers" list

Once have gone through all lines returned by Hough algorithm, I need to draw two lines for left and right lanes. For both lines y1 would be the height of image since we want to start from the bottom of the image. y2 for both lines would be slightly below the top of Area of Interest. I could set the y2 to half of the image but it would fail in more challenging situation when there is a curve on the road or camera is not positioned perfectly vertical. Now we need to find x1 and x2. For each left and right line:

* Don't draw anything if slopes list is empty
* Take average of slopes and centers lists
* Find x1 and x2 by formula ((y1-y)/averge_slope) + x) where x,y is center point calculated in previous step


Here is how the pipeline works:

#### Original
![alt text][orginal]
#### Yellow selection
![alt text][yellow]
#### White selection
![alt text][white]
#### Combining Yellow and White, Gaussian Blur
![alt text][yellow_white]
#### Canny Edge Detection
![alt text][canny]
#### Area of Interest Mask
![alt text][mask]
#### Raw Hough Lines
![alt text][hough]
#### Final output
![alt text][output]


### 2. Identify potential shortcomings with your current pipeline

#### Color
First problem I encountered was with the yellow color detection. In first iteration of my pipeline I started with a grayscale image and fed it to Canny Edge detection. It had having difficulty finding the yellow line. Then I switched to color based selection. For this I used HSV space because it is better able to cope with varying light levels. Then I applied inRange() selection. To figure out the color range I used hit and trial. 

This is still fragile because it can break on different brightness levels, camera models or variation in paint. I experienced this first hand when I applied the lane detector on my own video taken on Lahore Motorway where yellow point has different tone from the one used in USA. Day and night driving is a challenge.

#### Area of Interest
It is easy to define 'Area of Interest' on a straight road and per-determined camera height. If road is taking a tight curve or turn then we have to change the area to accommodate it. Similarly if there is a white or yellow car in front, it will confuse the lane detector, so we have to shrink the area we look for lane marking.
A fixed 'Area of Interest' will fail on cars of different heights and camera position. I experienced this first hand when I applied the lane detector on my own video where camera was slightly tilted towards ground.

#### Missing Lane Markers
We only work on a single image and don't take into account the fact that we found lanes lines in previous frame and it should be related current fame. Let's say if Lane markers is missing for a short while othen current logic fails to draw anything

#### Movement on Road
Cars change lanes, get off and on the freeway. Our alogothem assumes that car is always dring in a single lane.

#### Occlusion
If line of sight is not clear then current technique of finding lane lines will have problems.

### 3. Suggest possible improvements to your pipeline

* Deep learning algorithms are robust at coping with variation in color and other problems listed previously. We dont have to to hardcode HSV and Area of Interest with something like TensorFlow.
* We can use the knowledge that frames in a video are related and apply the knowledge form previous frame to better predict lanes in next frame. This can also fix jumpiness as seen in current implementation.
* We should separate out detection from tacking. Detect the lanes lines from Deep Learning and tack it with the specified techniques which are used to track features in a video.
