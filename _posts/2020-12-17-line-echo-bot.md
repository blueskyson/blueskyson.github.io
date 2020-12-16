---
title: "在 Heroku 上部屬 LINE Bot"
subtitle: ""
excerpt: "core dump"
layout: post
author: "blueskyson"
header-style: text
tags:
  - line bot
  - python
  - heroku
---

計算理論期末專題要做一個 line bot ，但是嘗試部屬的過程碰到許多困難，所以寫了這篇網誌紀錄下來。這個教學會部署一個用 python flask 實作的聊天機器人，使它在接收到文字訊息後，可回傳一模一樣的訊息。然後這次在 windows 平台開發，若要在其他系統請去參考別人的網誌。

## Step 1:

在 [LINE Developers](https://developers.line.biz/) 創建帳號:
![](https://raw.githubusercontent.com/blueskyson/image-host/master/line-echo-bot/1.png)
按下 **Create** 創建一個 Provider，我自己已經創建一個了，就沒有再建新的了
![](https://raw.githubusercontent.com/blueskyson/image-host/master/line-echo-bot/2.png)
接下來再點選 **Create a new channel** ，然後選 **Create a Messaging API channel**，就能輸入 Bot 的名稱、類別等，然後創建一個機器人
![](https://raw.githubusercontent.com/blueskyson/image-host/master/line-echo-bot/3.png)
最後會得到一個機器人長這樣:
![](https://raw.githubusercontent.com/blueskyson/image-host/master/line-echo-bot/4.png)

## Step 2:

- 創建 [Heroku](https://id.heroku.com/login) 帳號並下載 [heroku-cli](https://devcenter.heroku.com/articles/heroku-cli)
- 下載 [ngrok](https://ngrok.com/) 用以本地測試，如果不想在本地測試可以不用下載
- 下載 [python](https://www.python.org/) 或創建 python 虛擬環境，我自己是用 anaconda 的虛擬環境，版本是 python 3.6.7

## Step 3:

去 line 官方的 [line-bot-sdk-python](https://github.com/line/line-bot-sdk-python/tree/master/examples/flask-echo) 下載 echo bot 的範例程式碼和 requirements.txt，將他們放進你的 github repo
![](https://raw.githubusercontent.com/blueskyson/image-host/master/line-echo-bot/5.png)
然後用 pip 安裝 line-bot-sdk 和 flask 套件
```non
> pip install -r requirements.txt
```
接下來我們用 ngrok 連結本地端測試，首先我們需要設定跟 Bot 有關的環境變數，讓程式碼從環境抓取 Bot 的資訊，如此一來就不需要將 Bot 的 ID 暴露在程式碼中，降低資安風險。先回到 LINE Developers 查看 **Channel secret**
![](https://raw.githubusercontent.com/blueskyson/image-host/master/line-echo-bot/6.png)
點選 **Messaging API** 頁籤，issue 一個 **Channel access token**
![](https://raw.githubusercontent.com/blueskyson/image-host/master/line-echo-bot/7.png)
設定環境變數

```non
set LINE_CHANNEL_SECRET=c6548765346daf44929813e1236ade9f
set LINE_CHANNEL_ACCESS_TOKEN=ODWoxIDTCoxZ/2jxLRtqFhGxYDvtL2g2ZoPfceKv1AdSG+jtrE38CNY/Wz14Zj+NdJ4f5WMMi8re70KRuNRa0lFHxHYauoqzRPG9eX56zOFOUaA43Ebt3AjVOWKsxBB3o9dQjr9YUlbl9L0nUye69QdB04t89/1O/w1cDnyilFU=
```

打開一個 cmd 執行 ngrok ，將本地的一個 port 連結到一個 ngrok 的 server 作為 webhook ，預設應該是 8000 ，可以修改 app.py 改成其他 port

```non
> ngrok http 8000
```

cmd 會顯示以下資訊:

```bash
ngrok by @inconshreveable

Session Status                online
Session Expires               7 hours, 57 minutes
Version                       2.3.35
Region                        United States (us)
Web Interface                 http://127.0.0.1:4040
Forwarding                    http://97a5d3be3554.ngrok.io -> http://localhost:8000
Forwarding                    https://97a5d3be3554.ngrok.io -> http://localhost:8000
Connections                   ttl     opn     rt1     rt5     p50     p90 
                              0       0       0.00    0.00    0.00    0.00
```

請複製 ngrok 網址，貼在 LINE Developers 的 **Messaging API** 頁籤中的 **Webhook URL** ，網址後面需要補上 '\callback'
![](https://raw.githubusercontent.com/blueskyson/image-host/master/line-echo-bot/8.png)
在相同的頁籤中找到 **Auto-reply messages** ，按下 **Edit**，將回應模式設為**聊天機器人**
![](https://raw.githubusercontent.com/blueskyson/image-host/master/line-echo-bot/9.png)
回到 **Webhook URL** 的下方開啟 **Use webhook** (重要!! 不開的話 Bot 不會回應 webhook)
再開一個 cmd ，到 app.py 的目錄執行程式
```non
> python app.py
```
接下來將機器人加為好友，你輸入甚麼他應該就會回甚麼。如果機器人沒有正常運作，請檢查是否少做了甚麼步驟。

確認功能沒問題後，我們就能把程式推送到 Heroku ，不過現在時間很晚了，改天有空再更新吧!

## Step 4:
