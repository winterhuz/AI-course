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
