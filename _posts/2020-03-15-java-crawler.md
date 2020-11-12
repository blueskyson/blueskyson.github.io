---
title: "java 爬蟲教學"
subtitle: ""
layout: post
author: "blueskyson"
header-style: text
tags:
  - java
---

這篇教學會利用 jsoup 函式庫寫出一個爬取單一網頁並擷取網頁內所有文字的爬蟲，也算是一個筆記，以防自己將來需要用到這個程式的時候要 trace 半天才看得懂...

## 使用環境

Ubuntu 18.04  
openjdk version "1.8.0_242"  
jsoup 1.13.1

我沒有使用 maven 管理 java 套件，都是用手動管理的（之後學會用 maven 再補上 pom），所以這邊先上 [jsoup 官網](https://jsoup.org/download) 下載 jar 檔案，先將它放在這個專案的目錄內，然後就開始來寫程式

## 程式碼

```java
import java.io.IOException;

import org.jsoup.Jsoup;
import org.jsoup.nodes.Document;
import org.jsoup.nodes.Element;
import org.jsoup.select.Elements;

public class crawler {
    public static void main(String args[]) {
        System.out.println("Trying to connect to " + args[0] + " ...\n");
        String words = "", title = "";
        try {
            Document doc = Jsoup.connect(args[0]).get();    //connect to the link
            Elements elements = doc.body().select("*");     //select all elements in the website's body
            title = doc.title();                            //get the website's title
            for (Element ele : elements)                    //get all elements
                if (!ele.ownText().equals(""))              //if the text in a element is not ""
                    words += ' ' + ele.ownText();           //then add it to words
        } catch (IOException e) {
            e.printStackTrace();
        }
        System.out.println("fetch words from " + title + ':');
        System.out.println(words);
    }
}
```

## 程式流程

* `Document doc = Jsoup.connect(args[0]).get();` 是這個程式的主角，它負責向網址請求一個文件，然後回傳物件 `Document` 

* `Elements elements = doc.body().select("*");` 則是利用類似 javascript 的選擇器將網頁裡的元素解析出來，關於有哪些選擇的關鍵字，可以參考[這裡](https://jsoup.org/cookbook/extracting-data/selector-syntax)，我們在這個程式只需要使用 `"*"` 來選取所有網頁裡的元素

* 順代一提 `Elements` 這個物件很特別，它繼承自 *java.util.AbstractList* ，所以你可以注意到在 for 迴圈中我們用 `Element ele : elements` 來存取所有 `Elements` 裡的元素

* for 迴圈裡，我們將網頁中所有字串存入 `words` 並以空格隔開

* 最後印出所有網頁中的文字，程式結束

## 編譯執行

首先我們的檔案結構長這樣

```non
$ tree
.
├── crawler.java
└── jsoup-1.13.1.jar

0 directories, 2 files
```

這裡以爬取成大網站為範例

```non
$ javac -cp .:* crawler.java
$ java -cp .:* crawler  https://web.ncku.edu.tw/
fetch words from NCKU, 成功大學 National Cheng Kung University - NCKU, 國立成功大學 National Cheng Kung University:
跳到主要內容區塊 Menu Menu 在校學生 教職員工 畢業校友 未來學生 學生家長 國際學人 一般民眾 網站導覽 關於成大 教學 研究 行政 
招生 行事曆 中文 EN 本功能需使用支援JavaScript之瀏覽器才能正常操作 校長蘇慧貞致教職員工生信-協同開學防疫，共創安心校園 
more 成大導入智慧醫療　提升武漢肺炎臨床檢疫效率 more 成大水利系賴悅仁副教授實驗室　　國際頂尖期刊Nature專訪 more 氣候暖化蛾
知道　陳一菁教授研究成果登權威期刊Nature Communications more 從宿舍萌芽的夢想站上國際舞台　成大方程式賽車隊自組挑戰不可能 
more 成大「成蝶計畫」攜手國際企業打造人才育成平台 more ::: 故事．觀點 【成大醫院防疫前線紀事５】：醫護只是凡人，不是英雄，需
要大家團結配合 成大校友嚴瑞雄　號召口罩國家隊成軍 【影音】築起抗疫防線！ 衛教影片 沉著防備 在嘗試與意外中不斷前進　校友專訪斜槓
青年張志祺 more >> 成大快訊 成大醫學系李柏錦　高支持當選世界醫學生聯盟會長 【研發快訊】太空天氣預報系統發展與應用 – 陳佳宏助
理教授 做好準備守護成大！開學日全校動員齊防疫 防疫有「成」 成大主辦南區大專校院新冠肺炎防疫說明會 防疫安心就學　「線上學習錄播工
具操作」說明會 愛在疫情蔓延時！成大校友夫妻倆合贈500份維他命 通識課程成功學引領學生翻轉人生 疫中送暖！　台南阿霞飯店致贈成大
6600份米糕與甜粥 more >> 因應香港緊急情勢　成大具體措施 新型冠狀病毒資訊專區 近期活動 更多活動 全部公告 行政公告 學術研究 自
主學習 演講，研討會 社團藝文 招生公告 徵才公告 相關連結 ::: 701 臺南市東區大學路1號 TEL：06-2757575 回到頂部 OK Cancel
```

*`-cp .:*`* 代表將目前的目錄跟子目錄設為classpath，如果不想每次都設定classpath，可以在 ~/.bashrc 把程式會用到的路徑加進環境變數CLASSPATH

# 打包成jar檔

雖然這個程式成功的運作了，但是移到其他環境下又會發生classpath的問題，所以可以把所有用到的 class 都打包到 jar 檔

```non
$ mkdir Crawler
$ mv *.class Crawler/
$ jar -xf jsoup-1.13.1.jar
$ mv org/ Crawler/
```

此時檔案結構會長這樣

```non
.
├── crawler.java
├── jsoup-1.13.1.jar
├── Crawler
│   ├── crawler.class
│   └── org
│       └── jsoup
│           ├── Connection$Base.class
│           ├── Connection.class

. . .
```

然後把 Crawler/ 打包執行

```non
$ jar -cf Crawler.jar -C Crawler .
$ java -cp Crawler.jar crawler http://google.com
Trying to connect to http://google.com ...

fetch words from Google:
我們即將更新《服務條款》. 在新版條款生效 (2020 年 3 月 31 日) 前瞭解條款內容 查看 我知道了 Google 完全手冊 Google 商店 我
們即將更新《服務條款》. 在新版條款生效 (2020 年 3 月 31 日) 前瞭解條款內容 查看 我知道了 Gmail 圖片 登入 移除 回報不適當的
預測查詢字串 × 台灣 隱私權 服務條款 設定 搜尋設定 進階搜尋 你在 Google 搜尋中的資料 記錄 搜尋說明 提供意見 廣告 商業 搜尋服
務的運作方式
```
