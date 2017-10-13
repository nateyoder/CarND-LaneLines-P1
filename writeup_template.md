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
[scaled_image]: ./output_images/scaled_img.png
[unscaled_image]: ./output_images/unscaled_img.png

---

### Reflection

### 1. Describe your pipeline. As part of the description, explain how you modified the draw_lines() function.

The pipeline that I have created consisted of eight main steps. The first three steps were mainly simple 
transformations and filtering steps to prepare the image. The next step steps were used to find and filter line 
segments in the
 image using edges and the third step was to fit those line segments to form a single line.  Each of these steps 
 and their parameters will now be described in more detail.

As a first step in the analysis I created a simple function to non-linearly scale the RGB image using a for 
IMG<sup>exp</sup> where exp was chosen to be 4 in the 
current analysis. This was done based on the observation that in the challenge video the algorithm appears to do a 
very poor job of distinguishing the lane line edges on concrete and this transformation helped increase the 
difference between the lane lines and the concrete as seen below. 

![unscaled_image][unscaled_image]
![scaled_image][scaled_image]


In the second step the image was simply converted to grayscale to facilitate edge detection.

A gaussian blur was then applied to the image with a kernel size of 3.

In the fourth step Canny edge detection was applied to the image. A minimum threshold of 20 and a maximum threshold 
of 80 were found to work well even though the ratio between the two bounds was larger than that suggested by Canny 
in his paper.

In the fifth step the image was trimmed to the region of interest. This was set based on location of the lane lines 
in the current video and might change significantly with a wide-angle lens on the camera or other modifications to 
the video taping setup.

For the sixth step of the analysis I used the Hough transform to detect line segments. Because these segments were 
to be connected using a robust linear fit later in the pipeline I allowed rather short line segments to be detected.
 In the current analysis I used a pixel bin width of 2, a degree bin width of 2 degrees, set a density threshold of 
 20, and only required a max line length of 5 coupled with a max line gap of 1.
 
For the next step in the analysis I filtered these line segments based on their slope.  Because it was assumed that 
the car was between the lines for this project, the valid lane line segments on the left of the car should have a 
slope that is the opposite sign of the slope of the lane line segments on the right side of the car. In order to 
validate the line segments in a single comparision the slopes on the left side of the image were multiplied by -1.  
Then only the line segments that had slopes between 0.5 and 1.0 proceeded in the analysis.  One negative impact of 
this filtering process is that it seemed to remove most line segments due to the reflectors in between lane line 
dashes. This was deemed acceptable due to the amount of outliers that were also removed.

The last step in the pipeline turned all of the line segments into a single straight line. For this project I 
decided to approach this by actually drawing all of the line segments identified by the filtered Hough transform on 
a single image.  I then used the coordinates of the shaded pixels from this image to create a linear fit
for each lane line individually using a robust linear model.  This helps weight longer lines more while still taking 
into account all the line segments that are not considered outliers by the robust fitting algorithm. Several different 
linear models were investigated but a Huber regression model appeared to work relatively well for the videos 
investigated.  This model does not completely ignore outliers but weights them less in the fit. A 
relatively low epsilon value was chosen to count a relatively large percentage of the line segments as outliers and 
weight them less in the fit.

Because of the relatively low speed of the car and relatively good frame rate of the video the performance of the image processing pipeline outlined about can be significantly improved by taking 
into account the sequential nature of the video frames.  For this project a very simple approach was taken by using the pixels from the most recent Hough transform image and "activating" all of those pixels in the image that will be fit by setting those values to 1. The activated pixel's values are then linearly decreased to zero over 10 frames unless that pixel is activated again by another frame. In addition, these pixel activations can be used to weight each of the points in the robust linear fit so that older pixels have a smaller influence on the most recent fit. The impact of utlizing this temporal information significantly improved my performance on the videos.


### 2. Identify potential shortcomings with your current pipeline


One large drawback of my current pipeline is the amount of frame-to-frame variation that occurs in the videos.  
There are some frames where erroneously identified line segments negatively impact the fit.

Another limitation is the constrained and manually hard-coded region of interest.

Yet another large shortcoming of my algorithm is that it assumes that there are two lane lines, that the car is 
in between them, and that the slope of the lane lines is in a relatively small range.  Some of these assumptions 
will be violated anytime the car switches lanes and others are likely to be violated on curvy roads.

### 3. Suggest possible improvements to your pipeline

There are a large assortment of possible ways in which this algorithm could be improved. For instance, I am 
currently a very naive version of forgetting previous lane lines temporally which could likely be greater improved 
with a more sophisticated model.
 
Another improvement would be to relax the assumption that the lane lines are straight.  There are many ways in 
which this could be accomplished. An initial effort was performed to investigate using a polynomial fit to allow 
the lane lines to curve but it was found that this was subject to a lot of noise, even when a robust 
linear model was used to estimate the parameters. A piecewise linear fit might be another alternative.
 
Another improvement to this model might be to use a different line segment algorithm.  For instance, the line 
detection algorithm described in this [paper](https://scholar.google.com/scholar?q=A+simple+and+robust+line+detection+algorithm+based+on+small+eigenvalue+analysis&hl=en) 
might be interesting to investigate.  
 
I also look forward to learning about other potential approaches such as learning more about deep learning based 
attention models which might be able to estimate the probability that each pixel is part of a lane line boundary.
