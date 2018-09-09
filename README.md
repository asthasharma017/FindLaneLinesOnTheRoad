Steps:
When we drive, we use our eyes to decide where to go.  The lines on the road that show us where the lanes are act as our constant reference for where to steer the vehicle.  Naturally, one of the first things we would like to do in developing a self-driving car is to automatically detect lane lines using an algorithm.

draw_lines is the function for draws lines on the image. This function averages and extrapolates the lines based on the slope, determining wether the slope is positive or negative.

def draw_lines(img, lines, color=[255, 0, 0], thickness=5):
    """
    NOTE: this is the function you might want to use as a starting point once you want to 
    average/extrapolate the line segments you detect to map out the full
    extent of the lane (going from the result shown in raw-lines-example.mp4
    to that shown in P1_example.mp4).  
    
    Think about things like separating line segments by their 
    slope ((y2-y1)/(x2-x1)) to decide which segments are part of the left
    line vs. the right line.  Then, you can average the position of each of 
    the lines and extrapolate to the top and bottom of the lane.
    
    This function draws `lines` with `color` and `thickness`.    
    Lines are drawn on the image inplace (mutates the image).
    If you want to make the lines semi-transparent, think about combining
    this function with the weighted_img() function below
    """    
    right_slopes = []
    right_intercepts = []
    left_slopes = []
    left_intercepts = []
    left_points_x = []
    left_points_y = []
    right_points_x = []
    right_points_y = []
    
    y_max = img.shape[0]
    y_min = img.shape[0]
    
    for line in lines:
        for x1,y1,x2,y2 in line:
            slope = (y2-y1)/(x2-x1)

            #extrapolation lines on left
            if slope < 0.0 and slope > -math.inf:
                left_slopes.append(slope) 
                left_points_x.append(x1)
                left_points_x.append(x2)
                left_points_y.append(y1)
                left_points_y.append(y2)
                left_intercepts.append(y1 - slope*x1)
            
            #extrapolation lines on right
            if slope > 0.0 and slope < math.inf:
                right_slopes.append(slope) 
                right_points_x.append(x1)
                right_points_x.append(x2)
                right_points_y.append(y1)
                right_points_y.append(y2)
                right_intercepts.append(y1 - slope*x1)
            
            y_min = min(y1,y2,y_min)
            
    if len(left_slopes) > 0:
        left_slope = np.mean(left_slopes)
        left_intercept = np.mean(left_intercepts)
        x_min_left = int((y_min - left_intercept)/left_slope) 
        x_max_left = int((y_max - left_intercept)/left_slope)
        cv2.line(img, (x_min_left, y_min), (x_max_left, y_max), [255, 0, 0], 8)
    
    if len(right_slopes) > 0:
        right_slope = np.mean(right_slopes)
        right_intercept = np.mean(right_intercepts)
        x_min_right = int((y_min - right_intercept)/right_slope) 
        x_max_right = int((y_max - right_intercept)/right_slope)
        cv2.line(img, (x_min_right, y_min), (x_max_right, y_max), [255, 0, 0], 8)

Shortcoming:
1. In the canny edge detection algorithm we need to specify high and low threshold values mannually. Selecting wrong values for thresholds  can result in bad quality of detected edges.
2. This algorithm is not efficient when lane lines are discontinuous or not so clearly visible.

Possible Improvements:
1. One possible improvement for the first shortcoming can be improved by using mean or median of grey scale pixel values to get threshold values. This method is called mean value auto-thresholding if mean is used and median value auto-thresholding when median is used [1].
2. When the lane lines are discontinuous or not so clearly visible, the following methods:
Hough Probabilistic Transform, Bird eye view, weighted centroids, mean value theorem on Hough lines on curve, clustering of lines, Gaussian filter or shadow and illumination correction 
can be used [2].
 
 
References:
[1] http://www.kerrywong.com/2009/05/07/canny-edge-detection-auto-thresholding/
[2] https://airccj.org/CSCP/vol5/csit53211.pdf