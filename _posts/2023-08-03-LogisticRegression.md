---
layout: post
title: LogisticRegression
author: [Huz]
category: [self learing]
tags: [jekyll, ai, NN]
---

Neural Network 常常表示成 輸入->計算層->輸出  

我們就從單層計算層的網絡開始聊起，神經網絡到底是怎麼建構的

## 1. 網絡架構
神經網絡依據不同的目的會使用不同的算法來解決問題  
Logistic Regression 是一個線性回歸的變體構成的分類器  
主要用來分類二元問題  
  
  
當我們輸入一個圖片，想要知道圖片中是否有"貓"  
這個時候，輸入是"圖片" 也就是一個大小依像素而定的多維矩陣  
而輸出是一個 0或1 的常數，代表著"是否"的可能性  
  
要訓練神經網絡都會需要有一組訓練集  
假設已經有一組訓練集了，都包含了矩陣x輸入以及二元輸出y (0 or 1)

### 1.1 線性函式(linear function)
二元分類器最主要的目的就是劃出一條線分割兩區域


已知輸入 x 是一個矩陣， 輸出 y 是一個常數  
學過矩陣就知道，矩陣直至常數這中間必定經過降維  
我們設計一個矩陣參數 w 以及一個常數參數 b  
  
當 w * x 的時候會使得矩陣降維到 1x1 的大小，再加上參數 b 便是常數  
當然對於n個輸入 w * x 會使得矩陣降維到 nx1的大小，加上 nx1 矩陣參數 b 也會得出 n 個常數  

自此` z = w * x + b `

### 1.2 啟動函式(activate function)  
最終得出的常數希望是個機率
因此 z 還要經過啟動函式使其分布在0~1之間  
對於二元分類來說通常指的就是sigmoid函式  
當然還有像是tanh函式、等等  

自此 ` A = activate(Z) `  
  
### 1.3 成本函式(cost function)  
跑完映射函式得出結論後，會執行損失函式(L(A,y))計算與預期的誤差值  
已知值越接近誤差越小，而y只有(0,1)兩種可能，A則必定介於(0,1)之間  
而且，一般求平方差的函式不利於後續步驟  
  
這時我們提出一個滿足y在兩個狀態下皆可以表示誤差的函式  
  
`L(A,y) = - (ylogA + (1-y)log(1-A))  `

y=1時，A越靠近1就越大，使得-logA越小  
y=0時，A越靠近0就越小，使得-log(1-A)越小  
  
而所有訓練集的損失函式平均值便是成本函式(Cost)。  
  
`J(w,b) = 1/m * sum( L(A,y) )  `
  
  
### 1.3 梯度下降( gradient decent )  
神經網絡之所以引人嚮往便是因為最後的"學習"步驟  
這邊的思路是建立一個碗狀的函式，由Cost表示高度，w,b 表示位置  
初始時 w,b 會隨機設置在碗中任意位置，每一次回歸都令cost值順著"斜坡"滑向底部  
而這個斜坡的坡度也就是dJ/dw,dJ/db   
如此一來只要有足夠龐大且合適的訓練集，總會落在cost值最低的位置  
  
實務上也就是每一次回歸都更新一次w,b的值  
  w = w - learing_rate * dw  
  b = b - learing_rate * db  



## 2. 反向傳播演算法
  
結合以上的架構，我們可以來用數學總結一下  
從輸入w,b,x開始正向傳播  
  
`z = w * x + b` => `A =  1 /( 1 + e^(-z))` => `L = -(y*logA + (1-y)*log(1-A)) `

而回歸所需的計算過程因涉及到從結果溯源的偏微分，便稱為反向傳播
   
`dA = dL/dA = (-y/A) + (1-y)/(1-A) `  
`dz = dL/dA * dA/dz = da * d(sigmoid(z)/dz) = dA * A(1-A) = A - Y `  
`dw = dz * x = (A - Y)`  
`db = dz `  

  
  
  
  


