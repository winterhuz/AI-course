---
layout: post
title: faceDetect
author: [Huz]
category: [self learing]
tags: [jekyll, ai, opencv]
---

本周打算嘗試的就是OPENCV內建的人臉辨識模型

---
因為接著就要開始摸索神經網絡強化學習相關的東西了  
這兩周打算從"會使用模型"、"了解模型原理"開始著手  

  
進入到 `lib/python3.8/site-packages/cv2/data`    
可以看到裡面有大量的.xml文件，裡邊每一個都是預訓練的模型  
由文件名可以知道都是使用了HAAR特徵訓練而得，那haar特徵述算又是甚麼呢?  
## 1. haar特徵述算  
其實可以簡單地理解為一種近似的方法  
考慮了人的鼻子兩側有陰影顏色通常比中間要深  
眼睛有陰影比臉頰要深、嘴唇中間比上下要深等等特徵  
用符合特徵的黑白模板去檢索圖片中是否有地方符合  
再通過條件判定便能找出人臉。  
   
當然這其中有很多技術成分　  
### 1.1 矩形特徵  
最初始提出的haar共有三種特徵原型並由此構成矩形特徵   
而矩形特徵可以任意伸縮並套用在灰階圖片上的任意區域   
前面說了人臉有數個特徵模板像是鼻子眼睛等，那要怎麼判斷矩形特徵符合模板呢?  
這便是要看特徵值了，特徵值算法為矩形特徵的白色區域的像素和減去黑色區域像素和  
例如當一中心矩形特徵對到鼻子，特徵值就會計算白色（鼻子）的像素－黑色（鼻子兩側）的像素  
當然特徵值為了計算量會進行數值歸一化，也就是壓縮數值範圍  <br>
當特徵值在特定闕值內時便會判定符合模板<br>
<br>
### 1.2 積分圖 
已知矩形特徵可以投射在任意大小任意位置
且特徵值計算為白區減黑區<br>
若是使用　`for(白區內所有像素點):`　這類迴圈計算必然耗費巨大  
而積分圖只要在初始時遍歷一次從左上到所有像素點的矩形像素和  
接著要計算特定區域只需調用三個座標像素值即可完成

### 1.3 AdaBoost級聯分類器  
已知當矩形特徵的特徵值在特定闕值內時便會判定符合模板  
但符合上黑下白的不一定都是眼睛嘛，也有可能是桌腳（？  
因此為了增加準確度會事前整合出數個具有辨識度的模板串聯在一起  
當範圍內圖片符合眼睛模板，則判定是否符合鼻子，當符合鼻子則等等等  
一個級聯分類器是由許多強分類器樹狀構成，強分類器意味著與人臉具高度相關  
而每個強分類器又由許多弱分類器樹狀構成，弱分類器即可看成是上面所說的模板  
樹狀構成的緣故使得當某一判斷不通過就不會繼續耗費資源計算  
由此節省大量計算成本  
　　
## 2. 模型應用

首先匯入所需函式庫
    
    import cv2

將所使用的模型存至變數方便使用

    haar_front_face_xml = 'haarcascade_frontalface_default.xml'
    haar_eye_xml = 'haarcascade_eye.xml'

### 2.1 cv2.CascadeClassifier()

利用cv2.CascadeClassifier()建立該模型的分類器  

    cv2.CascadeClassifier("route")   
      route stands for 分類器的絕對路徑  
    可以使用cv2.data.haarcascades代替.xml檔的資料夾路徑  
再使用cv2.CascadeClassifier()的子函式detectMultiScale()辨識物體

    detectMultiScale(img, object, scaleFactor, minNeighbors, Flag, minSize, maxSize)
      img 欲用以辨識的圖片
      object       被檢測物體的矩形框(向量)
      scaleFactor  每次搜索的矩形框大小增長系數
      minNeighbors 構成成功檢測的最低鄰近矩形框個數
      Flag         四種分類器額外檢測的方式
      minSize/maxSize 限制成功檢定的目標區域大小
![官網連結](https://docs.opencv.org/4.x/d1/de5/classcv_1_1CascadeClassifier.html#aaf8181cb63968136476ec4204ffca498)

### 2.2 主程式
    def dynamicDetect():

        # 创建一个级联分类器 加载一个 .xml 分类器文件. 它既可以是Haar特征也可以是LBP特征的分类器.
        face_cascade = cv2.CascadeClassifier(cv2.data.haarcascades + 'haarcascade_frontalface_alt.xml')
        eye_cascade = cv2.CascadeClassifier(cv2.data.haarcascades + 'haarcascade_eye.xml')
        
        cap = cv2.VideoCapture('pexels-matthias-groeneveld-14691541 (2160p).mp4')
    
        while True:
            # 读取一帧图像
            ret, frame = cap.read()
            # frame = cv2.rotate(frame, cv2.ROTATE_90_COUNTERCLOCKWISE)
            if ret:
                frame = cv2.resize(frame, (0, 0), fx=0.2, fy=0.2)
                gray_img = cv2.cvtColor(frame, cv2.COLOR_BGR2GRAY)
    
                # 人脸检测
                faces = face_cascade.detectMultiScale(gray_img, scaleFactor=1.05, minNeighbors=4,
                                                      minSize=(15, 15))
                print(len(faces))
                for (x, y, w, h) in faces:
                    print(x,y,w,h)
                    # 在原图像上绘制矩形
                    cv2.rectangle(frame, (x, y), (x + w, y + h), (255, 0, 0), 2)
    
    
                cv2.imshow('Dynamic', frame)
                # 如果按下q键则退出
                if cv2.waitKey(1) & 0xff == ord('q'):
                    break

    cv2.destroyAllWindows()  
    if __name__ == '__main__':
        dynamicDetect()
## 3. 實作成果

![final](https://github.com/winterhuz/AI-course/blob/gh-pages/images/facedetect.gif)<br>
