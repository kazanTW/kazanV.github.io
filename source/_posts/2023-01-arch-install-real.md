---
title: Arch Linux 安裝紀錄 2023
date: 2023-01-30 00:08:17
categories:
- Linux distros
tags:
- Arch
---
## 前情提要
天啊上一篇竟然是兩個多月前……，我好爛喔嗚嗚嗚嗚。

經過幾個月的練習（好啦其實不需要那麼久，只是還在學期中我沒那麼多時間備份資料），趁著過年我終於做好準備，正式要跟陪伴我一年多的 Debian 說再見了。
（而且我是在除夕夜吃年夜飯時裝機，~~誰能比我瘋~~）

這次總體來說非常成功，沒有發生洗掉硬碟重來的情況，感天謝地QQ
以下就來簡單敘述過程吧w

## 設備資訊
Computer: Lenovo IdeaPad 5 14ALC05
CPU: 12 X AMD Ryzen 5 5500U

## 前置準備
首先，實體裝機就需要把安裝光碟映像檔（`.iso`）寫入硬體內，考量到今日筆電大多數已經沒有光碟機，都會推薦使用 USB 隨身碟（當然也可以用 SD 卡）；
`.iso` 檔一般都是用燒錄的方式寫入光碟，至於隨身碟，Windows 上有個好用的小工具 [Rufus](https://rufus.ie/zh_TW/)，可以讓你把 `.iso` 映像檔燒進隨身碟，
不過我當時已經是在使用 Debian Linux，所以直接用 `dd` 指令寫入就可以了：
```sh
sudo dd if=archlinux-x86_64.iso of=/dev/usb_location bs=1M
```
再來原本的家目錄資料要**備份**，因為當時裝 Debian 時我並沒有另外切一塊硬碟空間掛載家目錄，而安裝時會洗掉整個硬碟（建議），
這花了我兩天的時間才弄好……仔細一看才發現原來我家目錄肥的要死（Orz

一切做好準備，就該開始了！

## 裝機過程
插入安裝碟，從 UEFI 調整開機順序（USB 優先），重新開機就會進入安裝環境，
接著按照[安裝筆記](https://kazan.tw/posts/arch-install/)就能成功了！
不過需要注意以下事項：

### 網路連線
VM 會用 Host 的網路連線，所以不用特別設定，但是實機要先確認網路連線，
筆電一般都有無線網卡，可以利用 `iwctl` 指令確認網卡與 WiFi 熱點，並進行連線：
```sh
iwtcl device list                   // List all network device
iwctl station device get-networks   // List all available networks
iwctl station device connect SSID   // Connect 
```
不過我自己則是用 **USB 連接手機直接當作乙太網路連線**啦（真的是瘋子）。

### Microcode
如果使用 Intel/AMD 這兩家的 CPU，開機管理程式 GRUB 會需要處理器的微碼（microcode），所以需要安裝相關套件，
在開始安裝 GRUB 前，先下載處理器微碼套件：
```sh
pacman -S amd-ucode     // Install microcode
```
我使用 AMD 處理器，所以需要 AMD 的微碼套件 `amd-ucode`；反之若是使用 Intel 的就是裝 `intel-ucode`，
裝好之後再進行 GRUB 安裝與設定。

### 硬碟掛載
這次我決定要讓根目錄與家目錄分開掛載，所以就要按照安裝筆記的分割硬碟，
同時要注意掛載硬碟的掛載點（Mount Point）需要指定正確，建議掛載完可以用 `lsblk` 指令檢查；
我在第一次指定時就不小心把 `/` 的載點指到 `/boot` 上，導致安裝失敗（空間不足），
當我用 `lsblk` 檢查時，發現我的第一個硬碟區塊同時載了 `/` 跟 `/boot`，這個錯誤真的蠻蠢的……

### 輸入法
全部安裝好之後，中文輸入一開始是使用 RIME，但我後來發現自己還是習慣新酷音（Chewing）的注音輸入，
所以改安裝 `ibus-chewing`，同時要啟用輸入法工具。

### 圖形界面
因為用習慣了，加上這台筆電還算新，所以我就繼續沿用 KDE plasma：
```sh
pacman -Syu xorg-server plasma plasma-wayland-session kde-applications
```
所以我的圖形界面現在還長的一模一樣XDDD

另外，在調整輸入法時一直噴掉，意外讓我發現，我的 `sddm` 竟然正在使用 **Wayland**，
只好趕快登出，在登入界面選擇 **X11** 的 session，差點要沒辦法打字了……

### AUR Helper
AUR Helper 是什麼？請參考 Arch Wiki 的[說明](https://wiki.archlinux.org/title/Arch_User_Repository)（全英文，~~請自行翻譯~~）；AUR Helper 的選擇很多，我選擇 **yay**（但選自己喜歡的就好），
在安裝之前要先啟用 pacman 的 multilib 選項，
找到 `/etc/pacman.conf` 並用編輯器開啟，
將以下兩行取消註解（刪除 #）：
```
#[multilib]
#Include = /etc/pacman.d/mirrorlist
```
接著需要安裝 `git`（如果已經安裝可以跳過）：
```sh
sudo pacman -S git
```
然後將 yay 的原始碼 clone 到本地：
```sh
git clone https://aur.archlinux.org/yay.git
```
接著就可以編譯並安裝 yay：
```sh
cd yay
makepkg -si
```
**注意：AUR Helper 不要在安裝系統時以 root 身份安裝，最好在安裝好之後以 user 權限操作**

## 成果
現在這篇文章，就是在新的 Arch Linux 上打好的！
Thank you Debian，但是我要投向新的懷抱了（x

## 結論
~~不要學我想不開在除夕夜裝機（o~~
~~還有不要學我用手機網路還要插線（o~~

其實 Arch 裝機真的沒有很難，但就真的要習慣指令界面操作，不過我相信會用 Linux 的人肯定都會習慣的（？
新年新希望，祈禱換新的發行板能為我帶來新的未來。

新年恭喜，祝各位今年能夠更光明燦爛。

火山 / Kazan
2023.01.30