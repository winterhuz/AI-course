# OPENCV_PROjECT

剛好在網上看到一個道士憑空畫符的短影片，朱紅咒文循著道士的手指揮灑虛空    
特效挺不錯就想著試試看    


拆解一下，大概分成兩個步驟   

-辨識手指  
-文字隨手指出現並停留
    
    
第一點打算用簡單一點的顏色辨識來完成  
接著將影片分為無數揁，每一揁都辨識出手指的位置  
得出手指位置在畫面的座標後，在該座標上畫一個小圓  
隨著影片撥放，圓點連成線便成文字了。


----  

*  1.image transform
*  2.color identified
*  3.rail of pen, outcome and infix

In this project, the only package needed is "opencv-python"
install it at the "python package", then we can started.




# 1.image transform
First, here's some functions that is available to stretch pictures

    cv2.imread('img route')
"imgroute" stand for the complete route of the picture


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
