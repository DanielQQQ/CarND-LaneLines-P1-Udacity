# **Finding Lane Lines on the Road** 


My solution for this project can be found in the [Jupyter notebook](https://github.com/srinathravindan/CarND-LaneLines-P1-Udacity/blob/newCode/P1.ipynb).
The implementation works well on the 2 videos solidWhiteRight.mp4 [Input](https://github.com/srinathravindan/CarND-LaneLines-P1-Udacity/blob/newCode/test_videos/solidWhiteRight.mp4) [Output](https://github.com/srinathravindan/CarND-LaneLines-P1-Udacity/blob/newCode/test_videos_output/solidWhiteRight.mp4) and solidYellowLeft.mp4 [Input](https://github.com/srinathravindan/CarND-LaneLines-P1-Udacity/blob/newCode/test_videos/solidYellowLeft.mp4) [Ouput](https://github.com/srinathravindan/CarND-LaneLines-P1-Udacity/blob/newCode/test_videos_output/solidYellowLeft.mp4)

The implementation works on the challenge video as well. [Input](https://github.com/srinathravindan/CarND-LaneLines-P1-Udacity/blob/newCode/test_videos/challenge.mp4) [Output](https://github.com/srinathravindan/CarND-LaneLines-P1-Udacity/blob/newCode/test_videos_output/challenge.mp4)


## Lane Detection

The input to the code is a video of a car driving on a highway. We detect the lane lines in each frame/ image of the video, and create an output video. The detected lanes can be seen when the output video is played. 

The lane detection pipeline takes an image as input, performs a series of image processing operations and returns the result. The image input for the pipeline is a frame in the video input. The pipeline consists of the following steps. 

### Isolating the lanes
Assuming that the camera is placed at the cented of the car, and the car is approximately in the middle of the two lanes: one to the right and another to the left. The lanes are typically either white or yellow in color, and can be solid or broken lines. In addition, the paint on the lanes are not always full and complete. 
Here are a few sample images:

[image1]: ./test_images/whiteCarLaneSwitch.jpg
![alt text][image1]

Given these factors, we have to isolate the lanes from the rest of the image. The HLS (Hue Light Saturation) color scheme is more suitable than the RGB (Red Green Blue) scheme for identifying colors in an image. 

To detect white, it is sufficientt to filter on the Light component. I chose the values in the range ```[0,190,0]``` to ```[255,255,255]```. 
To detect yellow, I set the Hue component to be between 10 and 40, and the values are in the range: ```[10,0,100]``` to ```[40,255,255]```
The result of this step can be seen below:

[image2]: ./test_images_output/whiteCarLaneSwitchhlsimg.jpg
![alt text2][image2]

When the above image is combined with the greyscale image, the lanes become more prominent.

[image21]: ./test_images_output/whiteCarLaneSwitchywImage.jpg
![alt text21][image21]


Now that the colors yellow and white have been isolated, we must identify the yellow and whites lines. This is done by a series of image processing steps: .
- Edge detection using Canny Filter - kernel_size = 11, low_threshold = 50, high_threshold = 150. For kernel values less than 11 noisy lines were detected, and for values greater than 11, the filter was aggressively removing the lanes.
- Isolating the region of interest using a polygon - trapezoid with apex approximately halfway in the y direction (0.58 times y).
- Hough Transform for line detection - rho = 1, theta = np.pi/180, threshold = 20, min_line_length = 40, max_line_gap = 300. These settings ensure that even the faint lane lines are detected.

[image3]: ./test_images_output/whiteCarLaneSwitchedges.jpg
![alt text3][image3]

[image4]: ./test_images_output/whiteCarLaneSwitchmasked_edges.jpg
![alt text4][image4]

Hough transform returns several small left and right lanes satifying the conditions. There are several ways to smoothen out the lines and obtain a straight line to mark the lanes including (a) averaging overal several frames, and (b) finding an average line within a frame. I went with option 2. A small trick helps us obtain a single well defined straight line. Instead of averaging the (x,y) coordinates, I get the average slope of the right lane and the average slope of the left lane. With values corresponding to the bottom of image and halfway point as the values for y1 and y2, we can compute x1 and x2 using the slope and intercept. These values when plotted on the image would result in well defined straight lane lines. 

[image5]: ./test_images_output/whiteCarLaneSwitchresult.jpg
![alt text5][image5]

## Potential shortcomings
Several thresholds and filters in the approach are hardcoded, and with changes in conditions such as lighting, these may not work as desired. 
Further, the lane lines are jittery, introducing additional complexity in the process. 

