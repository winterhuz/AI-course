

import cv2   
import mediapipe as mp  
import imutils  

mpHands = mp.solutions.hands #mediapipe detect solutions
hands = mpHands.Hands()
mpDraw = mp.solutions.drawing_utils #mediapipe drawing solutions


def process_image(img):
    gray_image = cv2.cvtColor(img, cv2.COLOR_BGR2RGB)
    results = hands.process(gray_image)
    return results

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

