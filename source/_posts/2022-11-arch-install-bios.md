---
title: Arch Linux 安裝練習筆記 2022 - BIOS+MBR 
date: 2022-11-12 15:47:40
categories:
- Linux distros
tags:
- Arch
- VM
---
## 前情提要
上一篇竟然是上個月了……我果然怠惰了（跪

[上次](./2022-10-arch-install.md)練習裝機，因為目標主機用了近年主流的 UEFI+GPT，大部分東西都蠻好處理的；
但這次的機器很麻煩，竟然是類似傳統 DOS 機器的 **BIOS+MBR**（還是 2018 的機器，~~ASUS 真的可以吃大便~~），
所以很多步驟都需要更動，
而且那台的狀態更是慘烈，如果沒裝好可能就真的回不去了……
所以老樣子，用 VM 來練習裝吧！

※注意：本次安裝步驟大部分會沿用上次，只是部份步驟或設定參數有所不同，所以可以先參考上次的文章練習一次！

## VM 設定
跟上次一樣使用 Oracle VM VirtualBox，在新增 VM guest 的部份基本沿用上次的設定；
但有個不同的地方是，
System -> Motherboard 下面**不需要**選擇 "Enable EFI (special OSes only)"，
這樣才能製造出傳統 BIOS 開機環境。

## Arch 安裝
跟上次一樣，只有少部份步驟需要調整，
以下就列出需要更動的地方：

### 硬碟分割與格式化
BIOS 開機不需要 `/boot` 分區，所以分割時基本上只需要主硬碟跟 Swap 分區：
1. `/dev/sda1`：分給 Swap 的部份，我大概給了 2.7 GB；格式化一樣使用 `mkswap`。
2. `/dev/sda2`：剩下的空間都給主硬碟，一樣使用 Linux Filesystem，格式化使用 `mkfs.ext4`；另外這次我沒有分割根目錄與使用者家目錄。

### 設定套件來源
這次在嘗試時不知道為什麼 `reflector` 無法正確執行，
所以我手動編輯了 `/etc/pacman.d/mirrorlist`，
另外以下直接提供台灣的鏡像列表（僅使用 https 與 IPv4）：
```
Server = https://mirror.archlinux.tw/ArchLinux/$repo/os/$arch
Server = https://free.nchc.org.tw/arch/$repo/os/$arch
Server = https://archlinux.cs.nycu.edu.tw/$repo/os/$arch
Server = https://ftp.yzu.edu.tw/Linux/archlinux/$repo/os/$arch
```

### 安裝開機管理程式
本次一樣使用 GRUB，但安裝參數不太一樣：
```sh
grub-install --target=i386-pc
```
之後同樣生成開機設定檔即可。

# 然後就全部照抄上次（無誤）。

## 結論
是的你沒有看錯，實際上只有這些差異而已……
Arch 真的蠻好玩的www，~~還能讓我又水一篇文章（被打~~

總之，Arch 確實是一個很有趣的發行板，
有很多可以探索的地方，所以不如嘗試看看吧！