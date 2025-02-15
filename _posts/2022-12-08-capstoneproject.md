---
layout: post
title: capstone_project
author: [huz]
category: [Lecture]
tags: [jekyll, ai]
---

期末專題實作:
成員: 翁仲人、張易安 <br>
![](https://github.com/winterhuz/facenet-pytorch/blob/master/data/test_images/huz/1.jpg?raw=true)
<br>
<iframe width="743" height="418" src="https://www.youtube.com/embed/0cnXUcWJ4r4" title="Craziest Chat GPT Generated Stories" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>
<br>
<br>

---
## CHAT-GPT

### 系統簡介與功能說明

ChatGPT采用人类反馈强化学习（Reinforcement Learning from Human Feedback）训练而来，使用的方法与InstructGPT相同，但数据收集设置略有不同。

首先用有监督的微调训练一个初始模型：人类AI训练师提供对话，他们既扮演人类用户又扮演AI助手。

然后创建奖励模型，为了创建强化学习的奖励模型，需要收集对比数据，其中包括两个或多个按质量排序的模型响应。为了收集这些数据，需要进行AI训练师与聊天机器人展开对话，然后随机选择一个模型生成的消息并采样若干替代回答，由AI训练师对其进行排序。利用这种奖励模型，我们可以使用近端策略优化（Proximal Policy Optimization）对模型进行微调。这个过程需要经过多次的迭代。<br>
而經過層層訓練後的模型即為CHAT-GPT本身，在輸入端鍵入問題即可獲得回應。但本系統本身不具備絕對真實性，回應中可能出現虛假的答案或虛假的網址。<br>
————————————————
![](https://github.com/winterhuz/AI-course/blob/gh-pages/images/CHATGPTBLOCK.jpg?raw=true)
_posts/2022-12-10 languagedistinguish.md

### 系統方塊圖


### 製作步驟 
1.理解CHAT-GPT的運作原理

2.構思模擬目標

3.以文字組合試驗

**專題實作步驟:** 
<br>
1.確認實作目標:<br>
由本學期的教材網頁作為憑據，希望能生成一份進階課程的教材內容<br>
2.從源頭理解工具:<br>
經由老師提供的教材與搜尋所得了解相關發展歷史與所用技術<br>
3.調參訓練<br>
不同的用字與敘述方式都會影響回應是否符合期待<br>
4.建立網站<br>
依循回應所得人工建立完整網頁<br>
<br>
本小組成員約一周進行一次線上討論
並最後決定兩人各自選擇不同主題進行，再進行學術交流


---
## 系統測試與成果展示
---
<br>
經過多次思慮，最終將目標訂為在本學期教材的基礎上發展適合下一學期的銜接課程

在仔細觀察本學期教材之後給予指令，對於新學期的架構有以下幾個規則

1.教材內容不得與上學期重複

2.教材架構需與上學期雷同

3.教材內容需與本學期內容有關連性<br>
因未付費(沒錢)網站僅提供短篇幅的回覆，因此截圖較為零散

![](https://github.com/winterhuz/AI-course/blob/gh-pages/images/CHATGPT1.jpg?raw=true)
<br>
<table>
<tr>
<td><img src="https://github.com/winterhuz/AI-course/blob/gh-pages/images/CHATGPT2.jpg?raw=true"></td>
<td><img src="https://github.com/winterhuz/AI-course/blob/gh-pages/images/CHATGPT3.jpg?raw=true"></td>
<td><img src="https://github.com/winterhuz/AI-course/blob/gh-pages/images/CHATGPT4.jpg?raw=true"></td>
</tr>
</table>

架構出現後便再往深入引導。想產生的效果是如同老師的教材一般較少文案且能簡明扼要的呈現各式資訊。
<img width="476" alt="image" src="https://user-images.githubusercontent.com/100844817/212696063-879d5014-e980-4d22-9c77-6f4a7f61a9e2.png?raw=true">


接著經由人工製作產生的網頁如下<br>
[語音辨識](https://winterhuz.github.io/AI-course/example/2022/12/22/languagedis.html)
2022-12-22 languagedistinguish<br>

在探索過程中也發現chat-gpt可以用文字引導生成"利用linux語言完成特定任務"的內容。<br>
以上特點或許可以在未來幫助教師更快的達成目的。

本次期末作業的實作部分如上述所示，而剩下的部分將會是本人於寒假的自我規劃。利用本學期所學的技術更好的吸收新知，並一步步完成十四週的未來教材內容，並將無限制地提供給未來有需要的人使用。
<br>
<br>

*This site was last updated {{ site.time | date: "%B %d, %Y" }}.*

