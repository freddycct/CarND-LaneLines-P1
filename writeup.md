# **Finding Lane Lines on the Road** 

## Writeup Template

### You can use this file as a template for your writeup if you want to submit it as a markdown file. But feel free to use some other method and submit a pdf if you prefer.

---

**Finding Lane Lines on the Road**

The goals / steps of this project are the following:
* Make a pipeline that finds lane lines on the road
* Reflect on your work in a written report


[//]: # (Image References)

[image1]: ./examples/grayscale.jpg "Grayscale"

---

### Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

My pipeline consisted of 5 steps.
1. Convert image to grayscale
2. Apply Gaussian blur on grayscale image
   - kernel size of 3
3. Apply Canny Edge detection algorithm to obtain image with edges only
   - lower threshold = 50
   - upper threshold = 150
4. Apply a mask with the following coordinates, 
   - (0.05 * width, height)
   - (0.45 * width, 0.6 * height)
   - (0.55 * width, 0.6 * height)
   - (0.95 * width, height)
   - width, height = image.shape
5. Use the hough_lines function with the following parameters
   - rho = 2
   - theta = pi/180
   - threshold = 20
   - minLineLength = 40
   - maxLineGap = 20
6. Finally use weighted_img function to apply the output from hough_lines onto the original (colored) image.

In order to draw a single line on the left and right lanes, I modified the draw_lines() function by ...

1. From the set of lines obtained using HoughLinesP, 
   - add a normal distributed noise (mean 0 and std deviation 0.01) to the coordinates of those lines (to avoid infinite & nan gradient)
   - calculate the gradient = (y2-y1)/(x2-x1)
2. Add the gradient to a list, called gradients
   - if gradient is nan or infinite, ignore
   - if -0.5 <= gradient <= 0.5,
   - otherwise, add to list
3. Use Gaussian Mixture Model Clustering (setting number of clusters = 3)
   - the purpose of this is to cluster the gradients into 3 sets of (left lane, right lane, outlier)
   - sort the clusters according to their weights
   - choose the cluster with the highest weight, and take the mean as the first valid gradient, call this valid_gradient_1
   - find the next valid cluster by iterating through the sorted clusters, using mean as gradient such that we fulfill this condition: abs(gradient + valid_gradient_1) <= 0.3, 
4. Now that we have selected 2 valid gradients (clusters), it is time to find the intercept 'c' of a line: y=mx+c
5. Using the Gaussian Mixture Model, select the lines that belong to the selected clusters and derive c as an average of y-mx.
6. With m & c, we can find the start and end points of a line:
   - x1 = (h - c) / m
   - y1 = h
   - x2 = (0.6 * h - c) / m
   - y2 = 0.6 * h
   - where h is the height of the image
7. Then draw line (x1,y1), (x2, y2) on image      


### 2. Identify potential shortcomings with your current pipeline
1. Too many parameters to tune in the canny edge detection and hough lines transformation.
2. While the parameters can be optimized for images coming from one camera, other cameras may generate images that view the road from a slightly different angle. So the parameters used here will not generalize to other images from other cameras.
3. Too much domain knowledge is needed to tune the draw_lines() function using the clustering approach

### 3. Suggest possible improvements to your pipeline
1. If steering angle of the car is provided, it can be used as additional information to detect where is the lane line, assuming that the driver is always driving in the middle of the lane.
2. Since lane lines do not change gradient suddenly, it is possible to use the gradient in the previous image as a guide to the gradient in the current image. However, the design of this pipeline in this assignment treats each image independently, so I could not implement this idea.
