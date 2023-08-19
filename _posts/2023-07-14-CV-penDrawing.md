
---
layout: post
title: penDrawing
author: [Huz]
category: [self learing]
tags: [jekyll, ai, opencv]
---

剛好在網上看到一個道士憑空畫符的短影片  
朱紅咒文循著道士的手指揮灑虛空    
特效挺不錯就想著試試看    

---

## 1.大綱
拆解一下，大概分成兩個步驟   

-辨識手指  
-手指留痕  
  
手指估計還需要用到模型，先不碰  
實作這邊把手指改成筆頭，可以想像成有顏色的指套  

第一點打算用簡單一點的顏色辨識+辨識物描框來完成  
第二點試著土法煉鋼    
將影片分為無數揁，每一揁都辨識出筆頭的位置  
得出筆頭在畫面的世界座標後，在該座標上畫一個小圓  
隨著影片撥放，圓點連成線便成文字了  
  

### 1.1環境
首先環境配置了解一下  
/pycharm   
 
Python Package 安裝以下   
`/numpy 1.24.4`   
`/opencv-python 4.8.0.74`   



### 1.2 RGB & HSV

///先來科普一下  
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
///科普結束

## 2.辨識筆頭  

筆頭辨識這邊基本思路如下  
1. 將RGB圖像空間轉到適合cv的HSV空間
2. 找出辨識物的HSV參數範圍
3. 標記辨識物(筆頭)   
       
來 ~ 開電腦   
這次引用的函式庫有二，opencv的cv2函式庫及python numpy的 numpy  

    import cv2
    import numpy as np
    
### 2.1 RGB to HSV  
opencv函式在輸入圖片時不論是.png或.jpg檔  
輸入格式都是用的BGR空間  
而圖片要從RGB空間轉到HSV空間，有了cv2函式庫其實也就一段話的事  

    img = cv2.imread('連結.jpg')
    hsv = cv2.cvtColor(img, cv2.COLOR_BGR2HSV)

### 2.2 HSV range
接著找出我們要辨識的顏色範圍。  
具體操作是先建一個用來調整參數範圍的控制台    
用這控制台調整HSV三個參數的上下限找出符合的參數，藉此鎖定物體  

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

調整到mask的白色圖像契合所要辨識的物體即完成   
紀錄參數待之後使用  
  
這邊紀錄一下幾個基本語法  
        
        -cv2.imread('img route')
    "imgroute" stand for the complete route of the picture
        -cv2.resize(img, (x,y))
    "img" stands for your photo
    "x" stands for the width
    "y" stands for th height
        -cv2.resize(img, (0,0), fx=x, fy=y)
    "img" stands for your photo
    "x" stands for the rate of width, 1 as origin width
    "y" stands for the rate of height, 1 as origin height
        -cv2.rotate(img, rotatecode)
    "img" stands for your photo
    "rotatecode" stands for codes as "cv2.ROTATE_90_COUNTERCLOCKWISE","cv2.ROTATE_90_CLOCKWISE","cv2.ROTATE_180"   
### 2.3 mark the thing
目前為止已經可以辨識出特定的顏色，但圖片中可能會在別的地方也出現一樣的顏色  
可能是噪點或其他物體，這時可以用大小範圍來判定是否為辨識物  
順帶將辨識出的物體描邊並紀錄座標   

至此，筆頭辨識部分可以整合成以下兩個副函式。  

    def findPen(img):
            #顏色辨識
        lower = np.array([93, 56, 70])
        upper = np.array([157, 194, 229])
        hsv = cv2.cvtColor(img, cv2.COLOR_BGR2HSV)
        mask = cv2.inRange(hsv, lower, upper)
        result = cv2.bitwise_and(img, img, mask=mask)
            #獲取辨識物座標
        penx, peny = findContour(mask)
            #畫一實心圓標誌辨識物
        cv2.circle(imgContour, (penx, peny), 5, (253, 176, 192), cv2.FILLED)
            #紀錄辨識物位置座標
        if peny!=-1:
            drawPoints.append([penx, peny])  

    def findContour(img):
            #初始化
        x, y, w, h = -1, -1, -1, -1  
            #將辨識物描邊並記錄  
        contours, hierarchy = cv2.findContours(img, cv2.RETR_EXTERNAL, cv2.CHAIN_APPROX_NONE)  
            #對每個辨識物進行大小判定並近似成方框以利獲得辨識物座標  
        for cnt in contours:
            area = cv2.contourArea(cnt)
            if area > 10:
                peri = cv2.arcLength(cnt, True)
                vertices = cv2.approxPolyDP(cnt, peri * 0.02, True)
                x, y, w, h = cv2.boundingRect(vertices)
        return x, y


簡單解釋一下findcontour()的for迴圈是個啥  
>已知圖像是由像素構成，那麼三角形的外輪廓也將由邊上無數個點構成  
>如此多的輪廓點會極大的拖慢程式運行速度，因此我們會需要approxPolyDP()近似

順路帶上一個語法  
   
        contours, hierarchy = cv2.findcontours(img, mode, method)
    "img" stands for binary image(black and white)  
    "mode" stands for the way to search the contour.  
    "method" stands for the way to approximate the contour.  
    "contours" is a list of contour points,(n,x,y) as nst point and the coordinate.
    "hierarchy" is a matrix reveal the relationship among the contours.
    
詳情可看
[朝良大大的CSDN](https://blog.csdn.net/vclearner2/article/details/120776685)

## 3.筆頭留痕

如大綱所述，有了以上辨識出的辨識物座標  
加上連續輸出圓點的副函式  

    def draw(drawPoints):
            #將所記錄之辨識物座標軌跡皆畫上實心圓，實現筆跡
        for point in drawPoints:
            cv2.circle(imgContour, (point[0], point[1]), 15, (253, 176, 192), cv2.FILLED)

基本已經完成所有需要的功能，接著串起來就行  
開始撰寫主程式，當條件允許時順序呼叫副程式並循環，在特定條件下結束。  

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
  
## 4.實作成果  
成果如下
  
![](https://github.com/winterhuz/AI-course/blob/gh-pages/images/penDrawing_AdobeExpress.gif)  


缺陷可以說是相當巨大  
構成筆跡的圓點不夠密集無法形成柔順的筆畫  
下筆與離筆沒有對應的方式，導致仍有筆跡判定  
  
若圓點要更密集則需要更快的處理效能及更高的禎數  
成果中我減少筆跡判定的方式是加快移動速度，但更好的是消除判定  
像是直接遮擋筆頭或關閉判定程式，效果應該會好上不少  
  
不過這次主要是熟悉cv語法與處理圖片  
也確實熟悉很多，有所收穫    

期待實作到課堂，替換粉筆手與麥克墨水的那天  
  
  
   
