---
title: KDE Plasma vs GNOME —— 桌面環境使用心得
date: 2023-04-03 11:56:35
categories:
- Utils
tags:
- Linux
- DE
- KDE Plasma
- GNOME
---
## 前情提要
桌面環境用了一年多的 KDE Plasma，最近想說換換口味去玩一下 GNOME，
在踩到不少雷後也算是逐漸上手了，就來分享一下使用的心得，
~~順便懺悔自己又拖更。~~

## 桌面環境的發展歷史
如果是使用 Windows / macOS 的人可能比較不熟悉桌面環境這個概念。

桌面環境（Desktop Environment，DE），
是讓 *nix（這個名詞很常見，以後有機會來聊聊）類的作業系統可以有圖形界面的一個的重要系統套件類型。
以前 Linux 是純文字的作業系統，但後來為了方便操作，開發出了桌面環境讓 Linux 能夠像 Windows / macOS 一樣透過圖形界面操作系統。

DE 通常除了圖形界面外，還需要搭配視窗管理與顯示管理（Display Manager，DM）的工具，才能有堪用的視窗系統，
常見的 DE 如 GNOME、KDE Plasma、XFCE、i3 等。

## KDE Plasma
KDE Plasma 是屬於 KDE project 的一部分，KDE 是一個開源的圖形套件專案社群，其中 Plasma 是他們主力推廣的 DE 專案，
並且與 FreeDesktop.org 合作，可以使用他們釋出的小工具程式，通常還會配合 KDE 自家的系統應用程式，例如純文字編輯器 Kate、終端機模擬軟體 Konsole 等。

Plasma 預設使用的 DM 是 SDDM（Simple Desktop Display Manager），配合視窗管理器 kwin，預設界面類似 Windows 桌面（如下圖）。

### 優點
Plasma 最大的優點是優秀的**客製化（Customize）**，幾乎所有的東西都能按照自己的喜好與需求調整外觀、位置。
最簡單的例子，原本預設在下方的 task manager（就是那一條跟 Windows 很像的工作管理），可以放到左邊、上面、右邊；
另一個優點是大量的**小工具**，因為與 FreeDesktop.org 合作，大量的系統功能與資訊能夠透過小工具存取。

### 缺點
Plasma 主要的缺點是不那麼好看XD，因為主要是用 Qt 開發，加上設計比較簡樸，就會很像一些大學生自己幹出來的小玩具www
還有對 Wayland session 比較不友善，容易出 bug。
再來就是預裝的程式都是一堆 K 開頭，看了容易煩躁（？）

## GNOME
GNOME 最早是 GNU 計畫的一環，後來獨立成為 GNOME project，是不少主流發行版預設、甚至綁定的桌面環境（如 Ubuntu、Fedora 等）。
預設使用 GDM（Gnome Display Manager）作為 DM，GDM 同時內建視窗管理功能。
~~大概是因為美術設計比較認真~~，目前的 GNOME 看起來越來越像 macOS（如下圖）

### 優點
GNOME 的優點：**漂亮**。

這不是唬爛，GNOME 算是公認最漂亮的 DE，或許是因為使用 GTK+ 開發，線條上較為圓滑、流暢，
加上界面類似 macOS，按鈕比較簡潔，就營造出 GNOME 美觀大方的感覺。

### 缺點
缺點也很明顯：太美觀簡潔了（o）。
因為習慣把功能選單全部隱藏在右上角，要用的的時候就顯的麻煩不少，還要多按幾個按鈕。
還有因為預設使用 Wayland session，導致對 ibus 中文輸入的 bug 一直存在。
另外容易發生 icon missing，需要透過 GNOME tweaks（以前是 GNOME tweak tools）手動設定界面風格才能解決。

## 個人心得
先說結論，
1. 習慣 Windows 的先從 KDE Plasma 開始用
2. 習慣 macOS 的先從 GNOME 開使用

