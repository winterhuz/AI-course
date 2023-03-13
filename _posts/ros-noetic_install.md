安裝分為四步驟
1.環境 2.安裝 3.本地端環境 4.常見指令集

第一部分:<br>
獲取鏡像源、紀錄金鑰<br>
鏡像源的概念像是給予電腦一個路徑，有了相對應的鑰匙之後便可以沿著路徑下載該檔案<br>
這種大型的軟體常常都會使用<br>
鏡像源可以直接從官網提取，大家用中文查教程的話應該更常看到中國各大學提供的連結，畢竟...<br>
安裝官網鏡像源:<br>
sudo sh -c 'echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list'
給予金鑰:<br>


第二部分:<br>
再進行下一步驟之前務必先更新自己的狀態<br>
sudo apt update<br>

接著安裝檔案時要考慮是用甚麼環境，像是用ubuntu、anaconda等<br>
本人使用ubuntu的terminal配合python3安裝，最直覺的便是<br>
sudo apt install ros-noetic-desktop-full<br>
等待再等待，如果出現詢問框[Y/n]  鍵入大寫Y即可<br>
<br>
第三部分:<br>
這也是最多教程少寫的一段，而如果寫了通常都以"source操作"代指<br><br>

source /opt/ros/noetic/setup.bash<br>
echo "source /opt/ros/noetic/setup.bash" >> ~/.bashrc<br>
source ~/.bashrc<br>

目前為止最核心ros就已經搭建起來拉<br>
但要對ros操作還需要很多的合成指令集，與其以後慢慢加先來裝個常用的<br>

第四部份:<br>
下邊指令適用ubuntu20、ros-noetic以上版本<br>
更早版本可能會配套python3以前並需要python-rosdep之類的
常用:<br>
sudo apt install python3-rosdep python3-rosinstall python3-rosinstall-generator python3-wstool build-essential
<br>
接著最重要的一個:<br>
sudo apt install python3-rosdep
加載完後還需初始化<br>
sudo rosdep init
rosdep update

根據用途還會需要安裝夠多的指令庫或封包，只需按照以下格式<br>
sudo apt install ros-noetic-PACKAGE<br>
將PACKAGE改為需要的檔案即可<br>
<br>
這樣就可以正式開始ROS之旅了!<br>

