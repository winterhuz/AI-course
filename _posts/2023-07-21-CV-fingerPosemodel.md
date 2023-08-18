繼上次使用opencv內建的人臉識別模型後   
這次嘗試不同平台的模型，來看看使用上是否有落差  
  
## 1. Midiapipe Solutions
Midiapipe 是 Google Research 開發出的機器學習框架  
可接受各種像是音源、傳感器、視頻輸入  
MidiaPipe Solutions 提供了ML 的多種預訓練模型   
像是`midiapipe.solutions.hands``midiapipe.solutions.face_mesh`等等  
這次使用的是midiapipe的手部姿態辨識模型   
  
主要參考 ![求得大翻譯的 Midiapipe 手部辨識模型詳情](https://blog.csdn.net/weixin_43229348/article/details/120530937)  
   
這個模型的訓練過程挺有趣就拿出來講講  
為了訓練這個手指姿態模型他們先訓練了手掌的辨識模型  
主要由於三個原因  
1.手掌的邊緣更容易檢測  
2.手掌可用方形邊界框建模，減少錨點數  
3.手掌的目標與非極大值抑制算法較為契合  
辨識出手掌後才使用人工標注的樣本對手指關節位置進行回歸訓練   
也因為這些技術，在實作上可以看到儘管手指被手掌擋住了還是會有標點  
且預測的通常都蠻準確的，有些意外的欣喜    
  
那稍微了解模型原理之後就開始實際操作  

## 2. 模型應用  
一樣先匯入需要的函式庫

    import cv2   
    import mediapipe as mp  
    import imutils  

### 2.1 midiapipe.solutions.hands  
`midiapipe.solutions.hands` 是個會辨識出21個手部關節節點的模型，節點被稱之Landmarks  
  
`midiapipe.solutions.hands.Hands()` 則是模型參數的輸入點，共有四個參數  
`static_image_mode`調整是否為影片流輸入，True則啟用手部追蹤False則一禎禎搜尋  
`max_num_hands`調整畫面中有幾個手掌  
`min_detection_confidence`為辨識物標記為手掌的信心值範圍   
`min_tracking_confidence`為手部追蹤信心值範圍，與第一項掛勾  
在pycharm上可 `ctrl+lm` 函式檢視參數，若括號為空則為全數按初始值運行   
  
    mpHands = mp.solutions.hands
    hands = mpHands.Hands()
    mpDraw = mp.solutions.drawing_utils
### 2.2 手指姿態辨識  
初始化各項參數後便開始對圖片進行辨識  
  
`mp.solutions.hands.Hands().process(image)`  
此函數可將RGB空間的圖片進行手部姿態辨識  
返回一個structure，此處命名為result  
  
    def process_image(img):
        gray_image = cv2.cvtColor(img, cv2.COLOR_BGR2RGB)
        results = hands.process(gray_image)
        return results  

### 2.3 姿態描繪  
`results.multi_hand_landmarks` 是一個(n, 21, 3)陣列  
n為手掌個數，陣列包含21個landmarks的(x,y,z)  
(x, y)為座標值，(z)為該landmark與手腕節點的距離值  
  
`results.multi_hand_landmarks[n].landmark[m]`  
則為第n個手掌的第m個landmark的(x,y,z)  
  
`enumerate()` 是一個可以將數據轉為同時列出編號及數據的序列的函式  

    def draw_hand_connections(img, results):
        if results.multi_hand_landmarks:
              #標注每個手的各個landmark
            for handLms in results.multi_hand_landmarks:
                for id, lm in enumerate(handLms.landmark):
                      #將landmarks儲存的座標值轉為圖片內的絕對座標
                    h, w, c=img.shape
                    cx, cy =int(lm.x * w), int(lm.y * h)
                    print(id, cx, cy)
                      #紅圓標注landmarks並用drawing.utils繪製連結
                    cv2.circle(img, (cx, cy), 5, (0, 255, 0), cv2.FILLED)
                    mpDraw.draw_landmarks(img, handLms, mpHands.HAND_CONNECTIONS)
            return img
            
### 2.4 主程式

    def main():
        cap = cv2.VideoCapture('VID_20230717_183754.mp4')  
        while True:
            success, img = cap.read()
            if not success:
                print("not recieving")
                break
        #    img = cv2.resize(img, (500, 500))
            results = process_image(img)
            draw_hand_connections(img, results)
            cv2.imshow("hand tracker", img)
    
            if cv2.waitKey(1) == ord('q'):
                cap.release()
                cv2.destroyAllWindows()
  
    if __name__ == "__main__":
        main()

## 3. 成果

結果呈現: ![](https://github.com/winterhuz/AI-course/blob/gh-pages/images/handtracking.gif)   

這次使用Midiapipe模型直觀的感受到  
要使用別人的函式最直覺也最簡便的方式就是直接去看"官網範例"  
因為自己個性喜歡追根究底，看別人分享常常會有 "這函式啥意思"  
關於函式的輸入參數，輸出structure 果然還是要全盤了解比較心安  
