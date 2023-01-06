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


<iframe width="911" height="512" src="https://www.youtube.com/embed/zBu6nEFLTJE" title="2014台灣競速自走車國際賽第一名 台灣 趙師葦 36.618秒" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>
---
## CHAT-GPT

### 系統簡介與功能說明

ChatGPT采用人类反馈强化学习（Reinforcement Learning from Human Feedback）训练而来，使用的方法与InstructGPT相同，但数据收集设置略有不同。

首先用有监督的微调训练一个初始模型：人类AI训练师提供对话，他们既扮演人类用户又扮演AI助手。

然后创建奖励模型，为了创建强化学习的奖励模型，需要收集对比数据，其中包括两个或多个按质量排序的模型响应。为了收集这些数据，需要进行AI训练师与聊天机器人展开对话，然后随机选择一个模型生成的消息并采样若干替代回答，由AI训练师对其进行排序。利用这种奖励模型，我们可以使用近端策略优化（Proximal Policy Optimization）对模型进行微调。这个过程需要经过多次的迭代。
————————————————

-採自：https://blog.csdn.net/JarodYv/article/details/128159913

### 系統方塊圖


### 製作步驟 
1.理解CHAT-GPT的運作原理

2.構思模擬目標

3.以各式文字組合試驗

**專題實作步驟:** 



---
## 系統測試與成果展示
---

![](images/CHATGPT1.jpg?raw=true)

<br>
<br>
*This site was last updated {{ site.time | date: "%B %d, %Y" }}.*

