# Windows11 with ubuntu 20.04

### 首先 
雙系統跟虛擬機最大的不同就是雙系統本身並不影響電腦效能  
一個是一次用一個系統，一個是兩系統同時跑，效能不好顯而易見的  
雙系統最大的問題還是匹配性，建議多查查自己顯卡與ubuntu版本號能不能對上  

本人配置:

|本機端 | 欲安裝     | 顯卡                  | 
|------|------------|-----------------------|
|win11 |ubuntu20.04 | intel graphic iRIS xe |

---
正題開始阿  

配置雙系統共有三個步驟  
*磁碟配置  
*雙系統設置  
*雙系統下載  

## 一、磁碟配置 
雙系統與虛擬機基本都是在電腦上給你隔一區來搞耍  
只是雙系統要自己動手，現在來檢查自個的硬碟配置吧  
win11在主頁找 "電腦管理"-> "磁碟管理"
<img src="https://github.com/winterhuz/AI-course/blob/gh-pages/images/doublesystem_storage.png" width="600"/>  
雙卡單卡一目暸然，單卡就像我這樣只有一條，雙卡就會有兩條  
我一開始劃了兩個磁碟C:D:出來，挑一個大點來切個空間給雙系統吧  

 <img src="https://github.com/winterhuz/AI-course/blob/gh-pages/images/doublesystem_storage2.png" width="300"/> |<img src="https://github.com/winterhuz/AI-course/blob/gh-pages/images/doublesystem_storage3.png" width="300"/>  
這邊看自己要玩多大，我自己是想玩深度學習視覺辨識啥的要資料集，索性101GB充值下去  

