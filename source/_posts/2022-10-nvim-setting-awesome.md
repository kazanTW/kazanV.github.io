---
title: Neovim 設定 - 外觀篇
date: 2022-10-04 18:49:06
categories:
- Technical
tags:
- Neovim
- Editors
- Plug-ins
- Lua
---
## 前情提要
~~斷更快一週我很抱歉（~~

上次我們利用外掛設定了基本的 LSP 跟自動完成功能，
不過用著用著就想好像少了些什麼？

然後高秋看到就吐嘈我說：你為什麼直接用終端機界面，不設定個主題？
還有你不覺得有個狀態列的提示你工作會比較方便嗎？
我才恍然大悟，原來這種終端機界面的純文字編輯器還有這麼多有趣的外掛！

於是這兩天在一邊作業翻車的情況下，我又開始了設定~~（ㄊㄧㄠˊ ㄐㄧㄠˋ）~~之旅，
花了一整天終於搞定以下的「玩具」（？

以下就是一些簡單的紀錄。

## 背景主題
我選用了 [Nightfox](https://github.com/EdenEast/nightfox.nvim) 這個外掛，有很多主題選項，這邊我採用是淺色底的 Dayfox。
![Dayfox](https://user-images.githubusercontent.com/2746374/158456281-e8a968c0-e282-4943-b919-3d8454ca0529.png)
(Source: GitHub README)

## 狀態欄
我用了 [lualine](https://github.com/nvim-lualine/lualine.nvim)，然後改了一些顯示的設定讓他比較乾淨一點

## Tabline
其實 Neovim 並沒有真正的頁籤（tab），只是把 buffer 當成 tab 的顯示形式。
這邊使用了 [bufferline](https://github.com/akinsho/bufferline.nvim)，
接著就在這裡翻車了（幹

## 字體
結果我發現 lualine 跟 bufferline 出現了一些沒有正常顯示的字元，
查了 README 發現這些外掛使用了 [nvim-web-devicons](https://github.com/kyazdani42/nvim-web-devicons)，如果要正常顯示，本機上需要有安裝可用的 patched font，但我並沒有安裝，
後來就選擇了 [Nerd fonts](https://www.nerdfonts.com) 裡面的 Inconsolata，順便也把終端機跟 VS Code 的字體改了，
之後終於可以正常顯示了！\灑花/

## File Explorer
你以為純文字編輯器就沒有檔案瀏覽嗎？
[nvim-tree.lua](https://github.com/kyazdani42/nvim-tree.lua) 是你的好朋友www
![nvim-tree](https://github.com/kyazdani42/nvim-tree.lua/raw/master/.github/screenshot.png?raw=true)
(Source: GitHub README)

## 結語
蛤你說我沒教怎麼裝外掛？
給我去看 Neovim 設定的[前一篇](https://kazan.tw/posts/nvim-setting/)！

Neovim 還有很多好玩的外掛，我想我以後還有得研究了w

火山 / Kazan
2022.10.07