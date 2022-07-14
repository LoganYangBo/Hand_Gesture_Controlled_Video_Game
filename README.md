## Environment
- OS: MacOS El Capitan
- Platform: Python 3
- Librarys: 
	- OpenCV 3
	- appscript

## Demo Videos
- Youtube: [Click here](https://youtu.be/gfTRi-CNw_o)

## How to run it?
- run it in python
- press `'b'` to capture the background model (Remember to move your hand out of the blue rectangle)
- press `'r'` to reset the backgroud model
- press `'ESC'` to exit

# Hand Gesture Recognition Report

## Introduction
We will recognize gestures from video sequences and point out the number of fingers they show. To identify these gestures from a live video sequence, we first need to individually get the hand region by removing original background of the video sequence. After dividing the hand area, we count the number of fingers shown in the video sequence. 

## Subdivide the hand area
The first part for hand gesture recognition is to identify the whole hand area by eliminating original background of the video sequence. 

### -Background Obtaining
We control our system to look over 30 frames of a specific scene. And during this time, we calculate the running average of the current and previous frames. As a result, we get the background of the video sequence. And be careful that this process needs 5-6 seconds and do not move the camera during the whole period. 

### -Background Subtraction
After we figure out the background, we bring our hand in to let the system know that our hand is the new entry in the background, which means it becomes the foreground object. After calculating the background model by using the running average, we use the current frame to save the foreground objects (which is our hand) and the background. We calculate the absolute difference between the background model and the current frame (with our hand) to get the difference image that accommodates the newly added foreground object (which is our hand). 

### -Thresholds
To get the hand area from this differential image, we need to threshold this differential image so that only our hand area is visible and the background is painted black. Below is a image of all the steps above. 
![image](https://user-images.githubusercontent.com/36163586/178886911-66b1985b-29c5-4991-b8ec-826a6cf17ccf.png)
Fig.1. Flow diagram to get binary value image.

### -Contour Extraction
After thresholding the difference image, we find the contours in the generated image. The largest outline is thought to be our hands. So, you must make sure that your hand occupies the majority of the region in the frame. The result image is below. The following is the algorithm implementation process:NBD: starting from the boundary point (i, j), a boundary can be obtained by using the boundary tracking algorithm, and each newly found boundary B can be assigned a new unique number. NBD represents the number of the current tracked boundary.
<img width="202" alt="image" src="https://user-images.githubusercontent.com/36163586/178886943-e2eeb3fa-018f-4f42-b4fa-ade6a645668e.png">

Fig.2. The condition of the border following starting point (i, j) for an outer border (a) and a hole border (b). 


Raster scanning continues from point (i, j+1). End when the bottom right vertex of the image is scanned.

After completing the above steps, we take out all the pixels with largest number of same values as the contour of the hand. And in this situation, we choose all pixels with 2 and -2 value, then we get Fig.3. 
![image](https://user-images.githubusercontent.com/36163586/178886979-7b6b889c-9375-427f-9424-e873dc9e2680.png)
Fig.3. Find the hand according to the contour with the largest area. 

## Count the number of fingers
The second part for hand gesture recognition is to identify the number of fingers by analyzing the result image.

### -Find convex hull
The first step is to find the convex hull of the divided hand area and calculate the most extreme points (extreme top, extreme bottom, extreme left, extreme right) of the convex hull. We use cv2.convexHull() function to get the convex hull of divided hand area. The image is Fig.4. 
![image](https://user-images.githubusercontent.com/36163586/178887000-de765084-9028-413e-a020-3366b2a689cb.png)
Fig.4. Find convex hull
After finding the convex hull of the hand, we have two methods to count the number of fingers.

Method 1
### -Find extreme points and center point
We separately ascertain the locations of extreme top, extreme bottom, extreme left and extreme right points. Then we ensure the center point location of the palm by using these extreme points, as showed in Fig.5.

### -Draw circle
The third step is to construct a circle with the maximum Euclidean distance (between the center of the palm and the extreme points) as the radius. We use pairwise. Euclidean_distances() function from scikit-learn to calculate the four Euclidean distances. Then we form a circle with radius of 80% of the maximum distance, as showed in Fig.5.
 
### -Perform bitwise AND operation
The fourth step is to perform bitwise AND operation between divided hand area (frame) and circular ROI (mask). This reveals finger slices that can be further used to calculate the number of gingers displayed. We use cv2.bitwise_and function to compute the circular ROI. The above process is described in Fig.5.
![image](https://user-images.githubusercontent.com/36163586/178887020-ad18f062-5b4f-4e76-9edf-af905c61a943.png)
Fig.5. Flow diagram by using method 1.

### -Count fingers
The last step is to count the number of fingers. We try to find the bounding rectangle of each contour by using cv2.boundingRect function. And this return four number, where (x, y) is the upper-left coordinate of the rectangle and (w, h) is the width and height of the rectangle, as showed in Fig.5.In order to ignore the wrist, we can use the vertical (y) axis to check whether the (y + h) of the contour is less than the cy of the center point of the palm. The thumb of our hand usually can move horizontally or vertically. And in order to compensate it, we increase the cy by a factor of 0.25.Besides, we choose a limit as the perimeter*0.25, and check whether the number of contour points is within the limit. 

Method 2
From the process of convex hull, we can get the locations of each vertex of the convex hull, which is the locations of fingertips and wrist. Then we search the lowest point between two vertices, and compute the angle, as showed in Fig.6. We find all angles less than 90 degree, and treat them as fingers. However, we only use method 1 in our code. 
![image](https://user-images.githubusercontent.com/36163586/178887048-acee23d0-942d-404c-8953-195d0110ee37.png)
Fig. 6. Flow diagram by using method 2.

## An interesting application
After get the number of fingers, we decide an interesting dinosaur game. When the count of fingers is less than 3, the dinosaur will jump; Otherwise, it keeps walking, as showed in Fig.7. 
![image](https://user-images.githubusercontent.com/36163586/178887063-c03eda6b-4e21-40e2-b249-752314052cb5.png)
Fig.7. Dinosaur game controlled by hand.
