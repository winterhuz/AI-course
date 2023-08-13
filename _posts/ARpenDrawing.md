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




# 1.color detection

在電腦視覺中，所有的"顏色"都是用**特定參數組**表示的    
差別只在於使用的哪種空間而已    


直接上範例

![]()  

以上圖黃色而言
RGB空間以(255,255,0)標記 ; 而HSV以(60,100,100)標記


再來一張  
![]()  
  
這是比較黯淡的黃色  
RGB空間以(230,230,0)標記 ; 而HSV以(60,100,90)標記  
  
RGN空間的參數各自代表(紅，綠，藍) ; HSV空間的參數代表(色相，飽和度，明度)  
  
RGB在面對明度改變時三項皆會有牽連，而HSV則是單獨改動一數值  
因為同個物體拍攝角度不同時在畫面上通常會在**明度**上有明顯差異，所以通常使用HSV空間來處理圖像  


顏色辨識這邊基本思路如下  
1. 將RGB圖像空間轉到適合cv的HSV空間
2. 找出欲辨識物體的HSV參數範圍
3. 標記欲辨識物體

   
來 ~ 開電腦   
這次引用的函式庫有二，opencv的cv2函式庫及python numpy的 numpy  

    import cv2
    import numpy as np


說了嘛，圖片要從RGB空間轉到HSV空間，有了cv2函式庫其實也就一段話的事

    img = cv2.imread('連結.jpg')
    hsv = cv2.cvtColor(img, cv2.COLOR_BGR2HSV)

接下來的操作是建一個用來調整參數範圍的控制台  
在這控制台上可以調整HSV三個參數的上下限，藉此鎖定物體  

    cv2.namedWindow('Trackbar')
    cv2.resizeWindow('Trackbar', 640, 320)

    cv2.createTrackbar('Hue min', 'Trackbar', 0, 179, empty)
    cv2.createTrackbar('Hue max', 'Trackbar', 179, 179, empty)
    cv2.createTrackbar('sat min', 'Trackbar', 0, 255, empty)
    cv2.createTrackbar('sat max', 'Trackbar', 255, 255, empty)
    cv2.createTrackbar('val min', 'Trackbar', 0, 255, empty)
    cv2.createTrackbar('val max', 'Trackbar', 255, 255, empty)
    while True:
        hmin = cv2.getTrackbarPos('Hue min', 'Trackbar')
        hmax = cv2.getTrackbarPos('Hue max', 'Trackbar')
        smin = cv2.getTrackbarPos('sat min', 'Trackbar')
        smax = cv2.getTrackbarPos('sat max', 'Trackbar')
        vmin = cv2.getTrackbarPos('val min', 'Trackbar')
        vmax = cv2.getTrackbarPos('val max', 'Trackbar')
        print(hmin, hmax, smin, smax, vmin, vmax)

        lower = np.array([hmin, smin, vmin])
        upper = np.array([hmax, smax, vmax])
    
  
這樣一來就得到一個小小控制視窗拉   
再建兩個視窗用以可視化調整HSV參數範圍      
mask   以白色表示HSV範圍以內的圖像，黑色為範圍外       
result 經剔除HSV範圍後的彩色圖片    
    

        mask = cv2.inRange(hsv, lower, upper)
        result = cv2.bitwise_and(img, img, mask=mask)

        cv2.imshow('img', img)
        cv2.imshow('mask', mask)
        cv2.imshow('result', result)
        cv2.waitKey(1)

這邊紀錄一下幾個基本語法  
        
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
    
目前為止我們已經可以辨識出特定的顏色，但要精確辨識出位置還要經過一個步驟


    def findPen(img):
        #    img = cv2.resize(img, (0, 0), fx=0.05, fy=0.05)
        hsv = cv2.cvtColor(img, cv2.COLOR_BGR2HSV)
    
        lower = np.array([93, 56, 70])
        upper = np.array([157, 194, 229])
    
        mask = cv2.inRange(hsv, lower, upper)
        result = cv2.bitwise_and(img, img, mask=mask)
        penx, peny = findContour(mask)
        cv2.circle(imgContour, (penx, peny), 5, (253, 176, 192), cv2.FILLED)
    
        if peny!=-1:
            drawPoints.append([penx, peny])
1

    def findContour(img):
        contours, hierarchy = cv2.findContours(img, cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_NONE)
        x, y, w, h = -1, -1, -1, -1
        for cnt in contours:
     #       cv2.drawContours(imgContour, cnt, -1, (255, 0, 0), 2)
            area = cv2.contourArea(cnt)
            if area > 10:
                peri = cv2.arcLength(cnt, True)
                vertices = cv2.approxPolyDP(cnt, peri * 0.02, True)
                x, y, w, h = cv2.boundingRect(vertices)
    
        return x, y
    
    def draw(drawPoints):
        for point in drawPoints:
            cv2.circle(imgContour, (point[0], point[1]), 15, (253, 176, 192), cv2.FILLED)
        
    while True:
        ret, frame = cap.read()
    
        if ret:
    
            frame = cv2.resize(frame, (0, 0), fx=0.4, fy=0.4)
            imgContour = frame.copy()
            findPen(frame)
            draw(drawPoints)
            cv2.imshow('contour', imgContour)
        else:
            break
        if cv2.waitKey(20) == ord('q'):
            break
