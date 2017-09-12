# **Finding Lane Lines on the Road** 

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the images to figure out the segmented lines
* Try the pipeline on the video
* Improve the `draw_lines()` method to get the solid lines
* Try the improved pipeline on the optional challenge video
* Improve the `draw_lines()` method to fit the optional challenge video
* Reflect on your work in a written report


[//]: # (Image References)

[grayscale]: ./test_images_output/grayscale_solidWhiteRight.jpg "Grayscale"
[gray]: ./test_images_output/gary_solidWhiteRight.png "Gray"
[canny]: ./test_images_output/edges_solidWhiteRight.jpg "Canny"
[canny_masked]: ./test_images_output/edges_masked_solidWhiteRight.jpg "CannyMasked"
[result]: ./test_images_output/segmented_solidWhiteRight.jpg "Result"
[solid_result]: ./test_images/lined-solidWhiteRight.jpg "SolidResult"

---

### Reflection

### 1. Description of implement

My pipeline consisted of 5 steps:

1. I converted the images to grayscale with `grayscale()`, so i can get the result like:
![alt text][grayscale]
But the result is not gray, if i need the gary image, i need call `plt.imshow(gray, cmap='gray')` like:
![alt text][gray]

2. Then I define a kernel size and apply Gaussian smoothing, after tests, I choose 7 as my kernel size and call `gaussian_blur(gray,kernel_size)` method.

3. Now, I can define parameters for Canny and apply the 'edges = canny(image,low_threshold,high_threshold)'. So this time,
I choose 75 as low_threshold and 150 as high_threshold to get the result like:
![alt text][canny]

4. Then I get the region of interest, this image is size of `960*540` , so after analysing the image and the postion of the
lines. I get a polygone to get the region of interest `[(0,imshape[0]),(imshape[1]*0.499, imshape[0]*0.58), (imshape[1]*0.501, imshape[0]*0.58), (imshape[1],imshape[0])]`, like:
![alt text][canny_masked]

5. Finally define the Hough transform parameters and Run Hough on edge detected image. So the parameters that i got are
`rho = 2 theta = np.pi/180 threshold = 20 min_line_length = 25 max_line_gap = 10` with `hough_lines` and `weighted_img` methods, I can got the result with segmented lines like:
![alt text][result]

### 2. Improve the draw_line() to get the solid lines

To get the solid lines, we must get the whole line which cover the segmented lines.

So, first of all, I push all points into the arrays. But it exists some lines useless but was detected by the pipeline.
I must skip these lines. So I get the slope of every lines with `(y2-y1)/(x2-x1)`, when `slope < 0`, this is the left line and when `slope > 0`, this is the right line. For skipping the useless lines, I choose the lines between `> 0.5 (< -0.5)` and `< 0.8(> -0.8)`.

Now, I get all the points, I will get the average value of x1,x2,y1,y2 `mean({x})` to get the center line and its slope `(y2-y1)/(x2-x1)`. And the values of y1 and y2 are fixed, its the top position of line and the bottom position of line as `imshape[0]*0.58`. Now, we have slope, y value and average value, so we only need to and all the x1 and x2 with `x1_average + (y_top - y1_average) / average_slope` and `x2_average + (y_bottom - y2_average) / average_slope` to get the final solid lines like:
![alt text][solid_result]

### 3. Optinal Challenge

In this Challenge, the size of video has changed, so the position of line could not be good for the new video.

So, as a simple way to correct this problem, I passed the height of image in to the `draw_line()` methode to fit the images of all size. `line_y_top = int(imgHeight*0.58)`

### 4. Identify potential shortcomings with your current pipeline

One potential shortcoming would be what would happen when the sight of cars changed, maybe I cann't detect all lines in the site because the region of interest is fixed.

Another shortcoming could be the image is not smooth enough.

And the paremeters for each image treatment functions are not exact. So, there are still some useless lines.

### 5. Suggest possible improvements to your pipeline

A possible improvement would be to get a configuration parameter for my `process_image()` methode to pass the parameters like kernel_size, low_threshold, high_threshold, vertices, rho, theta, threshold, min_line_length, max_line_gap to customize the settings for each different videos. 

Another potential improvement could be to treat the edges of the solid line to make it much more smooth.

And also a potential improvement that we could separate the segmented lines methode and solid lines methode. Because in the real drive, we must know the shape of lines to decide if we can change lane. So the different situations are still usefull for the self-driving.
