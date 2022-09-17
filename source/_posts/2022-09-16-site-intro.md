---
title: 網站基礎架構
date: 2022-09-16 17:23:47
categories: 
- technical
tags: 
- infra
---

第一篇正式文章，來談談這個網站是怎麼產生的吧！

## 網域
網域一開始是想用 Freenom 的免費網域，但他們不給新註冊了QQ，
後來想既然我是台灣人，就用 `.tw` 的網域好了，然後就去跟 HiNet 買了（800 元好貴）。
後面 DNS 之類的就丟給 Cloudflare 託管了，還順便弄了自己的 mail address（~~真香~~）。

## 架站工具選擇
你說 WordPress？
CMS 是很方便啦但還要多設定 WordPress 需要的 domain 設定之類的，
嫌麻煩就決定用一般常用的 SSG。

至於 SSG 的選擇， 
最一開始想選擇 Hugo，但部署我覺得很麻煩就放棄了；
後來又改成 Django，再來又轉成 Vite + Vue3，但這兩個都是 App 式，檔案跟網站架構我有點無法理解，也就相繼放棄掉。
最後是看很多人推薦選擇了 Hexo，用下去驚為天人（？）

## 主題
就預設的 `landscape` 啊，還是大家有推薦的主題可以讓我參考www

## 部署
最方便的選擇就是 GitHub Pages 囉XD，~~雖然我也有 Netlify 之類的就是了~~。
而且大部分的 SSG 官方文件其實都有提供 GitHub Pages 的部屬流程，算是蠻友善的w

## 總結
大概就是，懶人設定？
雖然中間過程我也是被很多設定搞到唉唉叫，
（還發生過動到設定結果 `hexo g` 產生靜態檔案時渲染爆炸，所有頁面都一片白……）
不過最終成果還是很滿意的！

目前還需要改進的是：
1. 文章不夠多（我努力三天生一篇啦QQ）
2. 除了文章的頁面還可以更多樣一些
3. 要找人幫我作主視覺了，banner 是用預設的，favicon 甚至是拿 Google Material 的火山圖示wwww
4. 首頁的呈現我想改，但 `landscape` 的 layout 我有點看不懂，有大神願意教學嗎QQ

目前大概就這樣，想到再補充XD
這邊慣例附上各個工具的使用版本或是提供者：
- Hexo：6.3.0
- Hexo-cli：4.3.0
- Node.js：18.7.0
- Domain name provider：HiNet domain
- DNS hosting：Cloudflare
- Theme：Hexo-landscape

火山 / Kazan
2022.09.16