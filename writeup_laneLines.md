**Finding Lane Lines on the Road** 

(This is the write up report for this project. Readme.md is just a clone from the original repository)

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report

---

[//]: # (Image References)

[image1]: ./test_images_output/grayscale.JPG "Grayscale"

[image2]: ./test_images_output/gaussianSmoothing.JPG "Gaussian Smoothing"

[image3]: ./test_images_output/cannyEdge.JPG "Canny Edge Detection"

[image4]: ./test_images_output/roi.JPG "Region of Interest"

[image5]: ./test_images_output/edgeroi.JPG "Line Masked by Region of Interest"

[image6]: ./test_images_output/houghLine.JPG "Hough Line Detection"

[image7]: ./test_images_output/solidWhiteCurve_output.jpg "Solid White Curve Output" 

---

### Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

Once you read a sample image (or) a frame of the video, the pipeline consisted of 5 steps
1. Convert image to grayscale:
A number of computer vision algorithms use grayscale images as inputs so the features are color independent.

![alt text][image1]
       
2. Apply gaussian smoothing:
Here, we blur the image to smoothen the ‘weak’ edges.

![alt text][image2]
       
3. Apply Canny filter
This algorithm performs edge detection.

![alt text][image3]
       
4. Create an image mask for a region of interest
This is used to limit the lane finding to only a specific region in front of the car.

![alt text][image4]
      
![alt text][image5]
      
5. Apply Hough Transform
This algorithm helps us find ‘line’ like features in the image by transforming image space to a hough parameter space.

![alt text][image6]
	 
After this, we visualize the lanes on the original image. In this project, we focused constructing these high level steps to perform lane detection. So, a number of wrapper functions were provided making it easy to concentrate on the algorithm rather than the implementation.

![alt text][image7]

In order to draw a single line on the left and right lanes, I modified the draw_lines() as follows:

1. Averaging: Before I researched on ways to achieve the goal of drawing a single line for left and right lanes, this was the most intuitive idea I had. This consists of the following steps:
A. Use the two point slope formula to figure out if the points belong to the left or right lane. 
B. Average all the right points and left points separately to get a ‘mean’ left and right lane coordinates.
C. Draw lines using cv2.line.
Although this achieved the goal of drawing a single line for each of the lanes, the result was there was no easy way to extend these lines. I thought averaging would help with the noisy (jittery) nature of line detection and that did happen to some extent. However, the lines were still flickering. 

So I started researching online for inspirations. I came across the following 2 articles:
	
https://medium.com/@esmat.anis/robust-extrapolation-of-lines-in-video-using-linear-hough-transform-edd39d642ddf

http://ottonello.gitlab.io/selfdriving/nanodegree/python/line%20detection/2016/12/18/extrapolating_lines.html

2. Fitting lines: Based on the articles above, I started working towards fitting the points that belong to right and left lanes to separate lines. The algorithm consists of the following steps:
A. Use the two point slope formula to figure out if the points belong to the left or right lane.
B. Collect right and left lane points separately.
C. Feed these points to cv2.fitline function that uses least squares to find an approximation of a line for right and left each.
D. Use the resulting unit vector and x,y points to get the representative left and right lanes coordinates of our image.
E. Draw lines using cv2.line.
The result was much better. The lines were staying longer without too many jitters and were extending all the way from the bottom of the image to the top of the roi. There is still some wayward jitteriness especially in the solidYellowLeft test video.

### 2. Identify potential shortcomings with your current pipeline


1. We are assuming a hard coded region of interest. This may work for images/videos captured by this particular camera/mounting but probably not any other situation. 
2. The raw hough output is numerous and noisy. This is despite a very picky set of parameters to get over this. I relied too much on a setting a smaller roi so that hough lines detected outside this area would be rejected. 
3. Despite (2), we see some noise near the top of the roi.
4. The modified draw_lines() function is still not as stable as the example output video. 

### 3. Suggest possible improvements to your pipeline

1. We can take actual camera parameters (obtained through a calibration process) and extrinsics measurements (position of the camera) to be more intelligent about the region of interest construction.
2. We can post process the hough lines to be more intelligent about what to draw. This was achieved with the modification of the draw_lines() function as explained above.
3. Hough is just one method to find lines. We can look for parametric methods that can take the lane shapes into consideration and build models that way.
4. Probably some amount of averaging over frames could be used to make the lines more stagnant and less jittery. The first article in the links above talks about one such feedback mechanism.  

### Rubric Check
Have all project files been included with the submission?

Jupyter notebook – Done

Write up – Done

Does the pipeline for line identification take road images from a video as input and return an annotated video stream as output?
Done

Has a pipeline been implemented that uses the helper functions and / or other code to roughly identify the left and right lane lines with either line segments or solid lines?
Done

Have detected line segments been filtered / averaged / extrapolated to map out the full extent of the left and right lane boundaries? 
Done

Has a thoughtful reflection on the project been provided in the notebook?
Done

The optional part of the project has not been tried yet.

