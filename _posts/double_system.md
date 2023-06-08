# Windows11 with ubuntu 20.04

### 首先 
雙系統跟虛擬機最大的不同就是雙系統本身並不影響電腦效能  
一個是一次用一個系統，一個是兩系統同時跑，效能不好顯而易見的  
雙系統最大的問題還是匹配性，建議多查查自己顯卡與ubuntu版本號能不能對上  
另外一個影響的要素是系統BIOS模式，以下方法僅適用新式UEFI模式  
Win+R 輸入`msinfo32`  即可查看


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
win11在主頁找 "電腦管理"-> "磁碟管理"  ，或直接 win+X-> K

<img src="https://github.com/winterhuz/AI-course/blob/gh-pages/images/doublesystem_storage.png" width="600"/>  

雙卡單卡一目暸然，單卡就像我這樣只有一條磁碟0，雙卡就會有兩條  

雙卡單卡在切其實也是有講究的，像單卡要切最靠右的磁區  
雙卡則要切大概200M第一磁碟的C槽放開機程序，再切另一磁碟的最靠右磁區放剩下的資料  

我一開始劃了兩個磁碟C:D:出來，挑靠右的D:來切個空間給雙系統吧  
  :D  

 <img src="https://github.com/winterhuz/AI-course/blob/gh-pages/images/doublesystem_storage2.png" width="300"/> |<img src="https://github.com/winterhuz/AI-course/blob/gh-pages/images/doublesystem_storage3.png" width="300"/>  
 
這邊看自己要玩多大，我自己是想玩深度學習視覺辨識啥的要資料集，索性101GB充值下去  

>這邊解釋一下為甚麼要切最靠右的磁碟，因為Windows跟Ubuntu儲存格式其實不同  
>為了避免搜尋或存檔案出意外，保險一點直接物理隔離切成前後兩磁區，Ubuntu放在靠右也就是靠後的磁區


## 二、雙系統下載

雙系統下載有三類方式  

*虛擬機(WSL、VM)  
*wubi  
*USB式  

我這邊使用USB硬碟式安裝  
簡單來說就是把ubuntu載進usb裡邊，趁系統不注意，大力給他督進去  
當然Windows肯定也是有所防備，那我們就要慢慢讓他卸下心防  

首先來到  設定-> 隱私權與安全性-> 裝置加密   
一般家用版都是這個畫面，那如果你是專業版的就需要在同位置關閉bitlocker了  

<img src="https://github.com/winterhuz/AI-course/blob/gh-pages/images/doublesystem_safety1.png" width="400"/>  
                                                                                                            
把這個加密給關了，點擊關閉之後會開始讀條並顯示"可以繼續使用電腦..blalba"  
通常要等頗長一段時間，那我們就繼續操作  

主頁打開 控制台-> 硬體與音效-> 電源選項-> 系統設定(左方選擇"按下電源按鈕時的行為")  
把快速啟動關閉，這邊會這麼做是為了方便我們進入BIOS模式設定
                                         
<img src="https://github.com/winterhuz/AI-course/blob/gh-pages/images/doublesystem_safety2.png" width="600"/>   

這邊要去查一下自己電腦的BIOS模式的啟動按鈕，像hp就是F10，有些廠牌是F9或F12  
要進入BIOS模式首先要把電腦關了，在他開機的那段時間瘋狂點擊啟動鍵  
這邊分享我失敗多次的經驗總結，點擊重新啟動後會執行關機程序，一旦畫面上的文字消失了就是徹底關機準備開機  
這時候特別關鍵了，畫面會先亮一下隨即又暗下去，暗下去之後馬上點擊啟動鍵應該就成了，不行就多試幾次  
之所以這麼麻煩是因為hp的F10點進去只是個選單，BIOS設定是F10，boot manager是F9， 不小心多點或時間沒抓好就要重來...  

總之進入BIOS模式後要調整兩項，首先是security的 secure boot 設定為 Disable
再來


                                         





