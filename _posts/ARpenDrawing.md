# OPENCV_PROjECT

first of all, I'm usung opencv as the invironment  
using opencv cause less problem since the packages like python, can build inside of opencv


we can use simple function provided by opencv to complete this project
and I will cut it into 3 part to show you how to do it.

###  1.image transform
###  2.color identified
###  3.rail of pen, outcome and infix

In this project, the only package needed is opencv-python
install it at the "python package", then we can started.

## 1.image transform
First, I will show some functions that is available to stretch pictures

cv2.imread('img route')
"imgroute" stand for the complete route of the picture
put your picture in the same fold of your project file
then "img route" will be the name of the picture.


cv2.resize(img, (x,y))
"img" stands for your photo
"x" stands for the width
"y" stands for th height

cv2.resize(img, (0,0), fx=x, fy=y)
"img" stands for your photo
"x" stands for the rate of width, 1 as origin width
"y" stands for the rate of height, 1 as origin height

cv2.rotate(img, rotatecode)
"img" stands for your photo
"rotatecode" stands for 3 different code as "cv2.ROTATE_90_COUNTERCLOCKWISE","cv2.ROTATE_90_CLOCKWISE","cv2.ROTATE_180"

cv2.cvtColor(img,)
