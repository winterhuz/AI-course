繼上次使用opencv內建的人臉識別模型後
這次嘗試不同平台的模型，來看看使用上是否有落差

Midiapipe 是 Google Research 開發出的機器學習框架  
可接受各種像是音源、傳感器、視頻輸入  
MidiaPipe Solutions 提供了ML 的多種預訓練模型   
像是`midiapipe.solutions.hands``midiapipe.solutions.face_mesh`等等  
這次使用的是midiapipe的手部姿態辨識模型  
  
---
  
一樣先匯入需要的函式庫

    import cv2   
    import mediapipe as mp  
    import imutils  

`midiapipe.solutions.hands` 是個會辨識出21個手部關節節點的模型，節點被稱之Landmarks  
`midiapipe.solutions.hands.Hands()` 則是模型參數的輸入點，若空則為全數按初始值運行      
在pycharm上可 `ctrl+lm` 函式檢視參數  
  
`static_image_mode`調整是否為影片流輸入，True則啟用手部追蹤False則一禎禎搜尋  
`max_num_hands`調整畫面中有幾個手掌   
`min_detection_confidence`為辨識物標記為手掌的信心值範圍   
`min_tracking_confidence`為手部追蹤信心值範圍，與第一項掛勾  


    mpHands = mp.solutions.hands
    hands = mpHands.Hands()
    mpDraw = mp.solutions.drawing_utils

初始化各項參數後便開始對圖片進行辨識  
mp.solutions.hands.Hands().process(image)  
此函數可將RGB空間的圖片進行手部姿態辨識  
返回一個structure，此處命名為result  

    def process_image(img):
        gray_image = cv2.cvtColor(img, cv2.COLOR_BGR2RGB)
        results = hands.process(gray_image)
        return results

`results.multi_hand_landmarks` 是一個(n, 21, 3)陣列  
n為手掌個數，陣列包含21個landmarks的(x,y,z)  
(x, y)為座標值，(z)為該landmark與手腕節點的距離值  
  
`results.multi_hand_landmarks[n].landmark[m]`   
則為第n個手掌的第m個landmark的(x,y,z)  

`enumerate()` 是一個可以將數據轉為同時列出編號及數據的序列的函式  

  
def draw_hand_connections(img, results):
    if results.multi_hand_landmarks:
        for handLms in results.multi_hand_landmarks:
            for id, lm in enumerate(handLms.landmark):
                h, w, c=img.shape

                cx, cy =int(lm.x * w), int(lm.y * h)
                print(id, cx, cy)
                cv2.circle(img, (cx, cy), 5, (0, 255, 0), cv2.FILLED)
                mpDraw.draw_landmarks(img, handLms, mpHands.HAND_CONNECTIONS)
        return img

def main():
    cap = cv2.VideoCapture('VID_20230717_183754.mp4')

    while True:
        success, img = cap.read()
        if not success:
            print("not recieving")
            break
        img = cv2.resize(img, (500, 500))
        results = process_image(img)
        draw_hand_connections(img, results)

        cv2.imshow("hand tracker", img)

        if cv2.waitKey(1) == ord('q'):
            cap.release()
            cv2.destroyAllWindows()

if __name__ == "__main__":
    main()

