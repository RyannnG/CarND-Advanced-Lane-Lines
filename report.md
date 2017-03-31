## Advanced Lane Line

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

[//]: # "Image References"

[image1]: ./output_images/udis.png "Undistorted"
[image2]: ./output_images/undis3.png "Road Transformed"
[image3]: ./output_images/pipeline.png "Binary Example"
[image4]: ./output_images/warp1.png "Warp Example"
[image5]: ./output_images/warp22.png "Fit Visual"
[image6]: ./output_images/warp2.png "Output"
[image7]: ./output_images/line2.png "Output"
[image8]: ./output_images/line22.png "Output"
[image9]: ./output_images/lane2.png "Output"
[video1]: ./project_video_out.mp4 "Video"

## [Rubric](https://review.udacity.com/#!/rubrics/571/view) Points

###Here I will consider the rubric points individually and describe how I addressed each point in my implementation.  


### Camera Calibration

#### 1. Briefly state how you computed the camera matrix and distortion coefficients. Provide an example of a distortion corrected calibration image.

The code for this step is contained in the first code cell of the IPython notebook located in "Advanced Lane Line.ipynb" .  

I start by preparing "object points", which will be the (x, y, z) coordinates of the chessboard corners in the world. Here I am assuming the chessboard is fixed on the (x, y) plane at z=0, such that the object points are the same for each calibration image.  Thus, `objp` is just a replicated array of coordinates, and `objpoints` will be appended with a copy of it every time I successfully detect all chessboard corners in a test image.  `imgpoints` will be appended with the (x, y) pixel position of each of the corners in the image plane with each successful chessboard detection.  

I then used the output `objpoints` and `imgpoints` to compute the camera calibration and distortion coefficients using the `cv2.calibrateCamera()` function.  I applied this distortion correction to the test image using the `cv2.undistort()` function and obtained this result: 

![alt text][image1]

###Pipeline (single images)

#### 1. Provide an example of a distortion-corrected image.

To demonstrate this step, I will describe how I apply the distortion correction to one of the test images like this one:
![alt text][image2]
#### 2. Describe how (and identify where in your code) you used color transforms, gradients or other methods to create a thresholded binary image.  Provide an example of a binary image result.

The codes is in the section of "3. create a thresholded binary image" in the iPython notebook.

I used a combination of color and gradient thresholds to generate a binary image.  I apply sobel gradient on "x" direction and color thresholds on  channel "S" of HLS space. Here's an example of my output for this step.  

![alt text][image3]

####3. Describe how (and identify where in your code) you performed a perspective transform and provide an example of a transformed image.

The code for my perspective transform includes a function called `warp()`, which appears in  the "4. Apply a perspective transform" of the IPython notebook.  The `warp()` function takes as inputs an image (`img`), as well as source (`src`) and destination (`dst`) points.  I chose the hardcode the source and destination points in the following manner:

```
src = np.float32(
    [[(img_size[0] / 2) - 55, img_size[1] / 2 + 100],
    [((img_size[0] / 6) - 10), img_size[1]],
    [(img_size[0] * 5 / 6) + 60, img_size[1]],
    [(img_size[0] / 2 + 55), img_size[1] / 2 + 100]])
dst = np.float32(
    [[(img_size[0] / 4), 0],
    [(img_size[0] / 4), img_size[1]],
    [(img_size[0] * 3 / 4), img_size[1]],
    [(img_size[0] * 3 / 4), 0]])

```
This resulted in the following source and destination points:

|  Source   | Destination |
| :-------: | :---------: |
| 585, 460  |   320, 0    |
| 203, 720  |  320, 720   |
| 1127, 720 |  960, 720   |
| 695, 460  |   960, 0    |

I verified that my perspective transform was working as expected by drawing the `src` and `dst` points onto a test image and its warped counterpart to verify that the lines appear parallel in the warped image.

![alt text][image4]

![alt text][image5]

![alt text][image6]

#### 4. Describe how (and identify where in your code) you identified lane-line pixels and fit their positions with a polynomial?

The code is in the " 5. Detect lane pixels" section. Then I encapsulate it in the Lane class.

I first take a **histogram** along all the columns of the image. Then I  use a sliding window, placed around the line centers, to find and follow the lines up to the top of the frame. I place 9 window along the y direction with width of 100 pixels and set threshold of minimal number of pixels to 20.

After that, I was able to collect none zero pixels coordinates on both the left and right lane, then use `np.ployfit` to fit lanes lines with a 2nd order polynomial.

![alt text][image7]

![alt text][image8]

#### 5. Describe how (and identify where in your code) you calculated the radius of curvature of the lane and the position of the vehicle with respect to center.

Given a 2nd order polynomial, 

$f(y)=Ay^2+By+C$

The curvature could be computed via formulas like this:

$Rcurve=(1+(2Ay+B)^2)^{3/2}/|2A|$

I did this  in  "6. Determine the curvature". 

The y values of the image increase from top to bottom, so I use `np.max()` to measure the radius of curvature closest to the vehicle, 

#### 6. Provide an example image of your result plotted back down onto the road such that the lane area is identified clearly.

I implemented this step  in the function `pipeline()`.  Here is an example of my result on a test image:

![alt text][image9]

---

###Pipeline (video)

#### 1. Provide a link to your final video output.  Your pipeline should perform reasonably well on the entire project video.

Here's a [link to my video result](./project_video.mp4)

---

### Discussion

#### 1. Briefly discuss any problems / issues you faced in your implementation of this project.  Where will your pipeline likely fail?  What could you do to make it more robust?

The pipeline is still fragile and may fall on some situation where the lane isn't not clear on the road or there is a shade. And the robustness of  this approach highly depends on light and  what thresholds(color and sobel)  you choose. In the future, I wish to write codes that it could search for lanes and automatically tune the thresholds for best lane finding.

