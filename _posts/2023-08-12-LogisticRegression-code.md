
一樣先匯入需要的函式庫  
    import numpy as np
    import copy
    import matplotlib.pyplot as plt
    import h5py
    import scipy
    from PIL import Image
    from scipy import ndimage
    from lr_utils import load_dataset
    from public_tests import *

### 1.1 資料處理
對於輸入的圖形矩陣，通常會先做一層reshape處理  
當然首先要知道輸入的資料長什麼樣  

假設由某處匯入輸入包了
    train_set_x_orig, train_set_y, test_set_x_orig, test_set_y, classes = load_dataset()
接著就可以打印出來
    print ("train_set_x shape: " + str(train_set_x_orig.shape))
    print ("train_set_y shape: " + str(train_set_y.shape))
    print ("test_set_x shape: " + str(test_set_x_orig.shape))
    print ("test_set_y shape: " + str(test_set_y.shape))
得到
    train_set_x shape: (209,64,64,3)
    train_set_y shape: (1,209)
    test_set_x shape: (50,64,64,3)
    test_set_y shape: (1,50)

接著我們將其展開成一個2維矩陣，使每一列皆為一個展開的樣本
    train_set_x_flatten = train_set_x_orig.reshape(m_train,-1).T
    test_set_x_flatten = test_set_x_orig.reshape(m_test,-1).T

自此，我們已知訓練集維度是(12288,209)，測試集維度是(12288,50)
而 w 則應為(1,12288), b 應為(1,209)

### 1.2 正向傳播
前一篇文討論了此種演算法的正向傳播主要由三部分架構而成  
`z = w * x + b`  
`A = 1 / (1 + e^(-z)) `   
`J(w,b)= -1/m * sum(L(A,y)) = -1/m * sum( y*logA + (1-y)*log(1-A))`  

我們首先初始化參數w,b為零
    def initialize_with_zeros(dim):
    
        w = np.zeros((dim, 1), dtype = int)
        b = 0.
        
        return w, b
接著便可以使用已知的 w,b,X,Y 來完全建構正向傳播鍊了

def positive_propagate(w, b, X, Y):

    m = X.shape[1]
    // A = 1 / (1 + e^(-(w * x + b))) 
    A = 1 / (1 + np.exp(-(np.dot(w.T,X) + b))) 
    // J(w,b)= -1/m * sum(L(A,y))
    cost = -1/m * np.sum(Y * np.log(A) + (1-Y) * np.log(1-A))
    cost = np.squeeze(np.array(cost))
    
    return cost

### 1.3 反向傳播
同樣使用前篇文章所推導的公式    
def negative_propagate(w, b, X, Y):

    dw = 1/X.shape[1] * np.dot(X ,(A - Y).T)
    db = 1/X.shape[1] * np.sum(A - Y)
    grads = {"dw": dw,
             "db": db}

    return grads
### 1.4 迴歸更新

num_iterations表示訓練的次數，越多次越準確但通常呈現非線性關西
learning_rate表示cost在碗中下滑的距離，距離太長可能無法落在最低點，太短則耗時太久

copy.deepcopy()
>是copy函式庫的子函式，用以複製變數當下的狀態  
>是建立了一個新地址把原變數的內容拷貝過來的那種  
>也因此當變數為可變型別時不會隨之更動  

def optimize(w, b, X, Y, num_iterations=100, learning_rate=0.009, print_cost=False):
    w = copy.deepcopy(w)
    b = copy.deepcopy(b)
    
    costs = []
    for i in range(num_iterations):
            cost = positive_propagate(w, b, X, Y) 
            grads = negative_propagate(w, b, X, Y) 
            dw = grads["dw"]
            db = grads["db"]

            w = (w - (0.009 * dw))
            b = (b - (0.009 * db))

            if i % 100 == 0:
                costs.append(cost)
        
                if print_cost:
                    print ("Cost after iteration %i: %f" %(i, cost))
    
    params = {"w": w,
              "b": b}
    
    grads = {"dw": dw,
             "db": db}
    
    return params, grads, costs
### 2.1 預測

先前訓練過程中我們把預測值A與實際值y做比較   
如今訓練大成，是時候讓 A 來判斷了  
    def predict(w, b, X):
        m = X.shape[1]
        Y_prediction = np.zeros((1, m))
        w = w.reshape(X.shape[0], 1)
    
        A = 1 / (1 + np.exp(-(np.dot(w.T, X) + b)))
        for i in range(A.shape[1]):
            if A[0, i] >0.5:
                Y_prediction[0,i] = 1
            else:
                Y_prediction[0,i] = 0
    return Y_prediction
此時 Y_prediction 內部為一排0101的陣列

### 2.2 整合

    def model(X_train, Y_train, X_test, Y_test, num_iterations=2000, learning_rate=0.5, print_cost=False):
    w, b = initialize_with_zeros(X_train.shape[1])
    params, grads, costs = optimize(w, b, X_train, Y_train, num_iterations=100, learning_rate=0.009, print_cost=False)
    w = params["w"]
    b = params["b"]
    Y_prediction_test = predict(w, b, X_test)
    Y_prediction_train = predict(w, b, X_train)
    if print_cost:
        print("train accuracy: {} %".format(100 - np.mean(np.abs(Y_prediction_train - Y_train)) * 100))
        print("test accuracy: {} %".format(100 - np.mean(np.abs(Y_prediction_test - Y_test)) * 100))

    
    d = {"costs": costs,
         "Y_prediction_test": Y_prediction_test, 
         "Y_prediction_train" : Y_prediction_train, 
         "w" : w, 
         "b" : b,
         "learning_rate" : learning_rate,
         "num_iterations": num_iterations}
    
    return d
