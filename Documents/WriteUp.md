##Writeup Template
###You can use this file as a template for your writeup if you want to submit it as a markdown file, but feel free to use some other method and submit a pdf if you prefer.

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

[image1]: ./Camera-Calibration-Images/calibration2.jpg "Distorted"
[image2]: ./Camera-Calibration-Undistorted-Images/Undistorted2.jpg "Undistorted"
[image3]: ./Output-Images/FinalEdgeDetection1.jpg "Binary Example"
[image4]: ./Output-Images/BirdEyeView2.jpg.jpg "Warp Example"
[image6]: ./Output-Images/AdvancedLaneDetection1.jpg"Output"
[video1]: ./project_video.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points
###Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  

---
###Writeup / README

####1. Provide a Writeup / README that includes all the rubric points and how you addressed each one.  You can submit your writeup as markdown or pdf.  [Here](https://github.com/udacity/CarND-Advanced-Lane-Lines/blob/master/writeup_template.md) is a template writeup for this project you can use as a guide and a starting point.  

###Camera Calibration

####1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.
The code for this step is contained in the first and second code cell of the IPython notebook located in "Python-Files/Udacity-Advanced-Lane-Finding.ipynb"
I loaded the chessboard image from the dataset and used `cv2.findChessboardCorners()` to detect chessboard corners. I assume the chessboard is fixed on the (x, y) plane with z=0, such that the object points are the same for the calibration image. Thus, `Objpoints` is just a replicated array of coordinates for all chessboard corners in a test image. Whereas `ImgPoints` are with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection. 
I then used the output `ObjPoints` and `ImgPoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result as shown in the .ipynb file
Details for OpenCV APIs be seen in http://docs.opencv.org/2.4/doc/tutorials/calib3d/camera_calibration/camera_calibration.html
![alt text][image1]

###Pipeline (single images)

####1. Provide an example of a distortion-corrected image.
The code for this step is contained in the second code cell of the IPython notebook located in "Python-Files/Udacity-Advanced-Lane-Finding.ipynb"
The code cell provides an example of undistortion of a chessboard image. It also shows code for reprojection error.
![alt text][image2]

####2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.
The code for this step is contained in the third-ninth code cell of the IPython notebook located in "Python-Files/Udacity-Advanced-Lane-Finding.ipynb".
First, I defined a region of interest (ROI). 
I defined functions to create threshold binary image based on absolute threshold, magnitude and gradient of sobel operator; absolute threshold of hue and saturation.
Then I defined an edge detection function that first implements a gaussian blur and then use a combination of the above mentioned functions and look for edges in the ROI.
Here's an example of my output for this step. 
![alt text][image3]

####3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.
The code for this step is contained in the twelfth-fourteenth code cell of the IPython notebook located in "Python-Files/Udacity-Advanced-Lane-Finding.ipynb".
The code for my perspective transform includes a function called `cv2.warpPerspective()`. I first undistorted the image, then defined road vertices object points and image points. Here, it is assumed that the road is a planar surface throughout. The `cv2.warpPerspective()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points. 
I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.
The follwing link details on how we can get accurate perspective projection: http://stackoverflow.com/questions/15768651/generating-a-birds-eye-top-view-with-opencv

####4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?
The code for this step is contained in the fifteenth-twenty first code cell of the IPython notebook located in "Python-Files/Udacity-Advanced-Lane-Finding.ipynb".
Once, I have image for both the left and right lanes from above, then I use the Numpy polyfit() function to fit a polynomial equation to each lane. I also calculate the Radius of Curvature for each lane at this stage.
Once, I have the warped binary image, I extracted the actual lane pixels for both the left and right lanes from the warped binary image.
I define a function to generate a histogram of the bottom half of the image. Then using the two peaks in this histogram, I determine a good starting point to start
searching for image pixels at the bottom of the image.
Once these points are calculated, I perform the same operation for a sliding window. 
I use `polyfit()` function to find the polynomial curve that defines the left and right lanes.
Once these have been calculated, I do some checks to determine if the calculated lane if ‘satisfactory’ or not. I check for the distance between the two lanes to see if it is appropriate and also the radius of curvature. 
Finally, I draw the detected lanes back on to the undistorted image.

####5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.
The code for this step is contained in the sixteenth-seventeenth code cell of the IPython notebook located in "Python-Files/Udacity-Advanced-Lane-Finding.ipynb".
I use `polyfit()` function to find the polynomial curve that defines left and right lanes while considering the lane can be defined by a second-order polynomial curve.
Then I compute radius of curvature from this line using the first order and second order differentials from the resultant polyline. The tutorial gives step-by-step explanation
http://www.intmath.com/applications-differentiation/8-radius-curvature.php
The position of the vehicle is determined by finding the difference between the center of the image and difference between the left and the right lane points.


####6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.
Here is an example of my result on a test image:
![alt text][image6]

---

###Pipeline (video)

####1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video (wobbly lines are ok but no catastrophic failures that would cause the car to drive off the road!).
The video can be found in Videos/Output.mp4

---

###Discussion

####1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?
I initially failed to understand how to warp the image to bird view projection since I did not have any actual points on the road that would correspond to image points. I saw the example from the videos and performed it via hit and trials. The implementation is likely to fail when the lane width changes since I have used the fixed ROI. I would like experiment more with an adaptive ROI. 
The implementation might fail in different illumination conditions, and also at sharp turns. The solution can be improved if ROI is adaptive. Also, it might improve if we could use adaptive thresholds for different weather and light conditions.

