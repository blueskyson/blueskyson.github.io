---
title: "肺炎疫情 (COVID-19) 到 2020 年 12 月的現狀 (自 Kaggle Notebook 翻譯)"
subtitle: ""
excerpt: "COVID-19 python data science"
layout: notebook
author: "blueskyson"
header-style: text
mathjax: true
tags:
  - data science
  - python
---


來源: [https://www.kaggle.com/corochann/covid-19-current-situation-on-december/notebook](https://www.kaggle.com/corochann/covid-19-current-situation-on-december/notebook)  
此筆記的資料集來源: [Johns Hopkins University github repository](https://github.com/CSSEGISandData/COVID-19)  
最後更新: 2021-02-01

## 主要重點

1. [**綜觀全球疫情走勢**](#綜觀全球疫情走勢)
    - 以 `折線圖` 分析
2. [**分析各國確診與死亡人數變化**](#分析各國確診與死亡人數變化)
    - 以 `柱狀圖`、`顏色指標全球動態地圖`、`顏色指標折線圖`、`顏色指標熱圖` 分析
3. [**各國省分詳細資訊**](#各國省分詳細資訊)
    - 有多少國家提供精確的省分疫情資料
    - 以 `Dictionary` 分析
4. [**深入探討美國疫情**](#深入探討美國疫情)
    - 以 `表格`、`顏色指標美國動態地圖`、`顏色指標折線圖`、`顏色指標熱圖` 分析
5. [**分析歐洲疫情**](#分析歐洲疫情)
    - 以 `顏色指標歐洲動態地圖`、`顏色指標折線圖`、`顏色指標熱圖` 分析
6. [**分析亞洲疫情**](#分析亞洲疫情)
    - 以 `顏色指標亞洲動態地圖`、`顏色指標折線圖`、`顏色指標熱圖` 分析
7. [**哪些國家已經脫離疫情高峰**](#哪些國家已經脫離疫情高峰)
    - 以 `顏色指標全球動態地圖` 分析
8. [**全球疫情何時趨緩**](#全球疫情何時趨緩)
    - 使用 sigmoid 函數預測
    - 以 `顏色指標折線圖` 分析

### 安裝

pip install -r requirements.txt

{% include notebooks/COVID-19-current-situation-on-December/1.html %}

### 載入資料

使用到的資料集: [COVID-19/csse_covid_19_data/csse_covid_19_time_series/](https://github.com/CSSEGISandData/COVID-19/tree/master/csse_covid_19_data/csse_covid_19_time_series)

- 全球
    - `confirmed_global_df` : dataframe，儲存從 2020/1/22 到現在的各國確診人數

    - `deaths_global_df` : dataframe，儲存從 2020/1/22 到現在各國死亡人數

    - `recovered_global_df` : dataframe，儲存從 2020/1/22 到現在各國康復人數
- 美國
    - `confirmed_us_df` : dataframe，儲存從 2020/1/22 到現在美國確診人數
    
    - `deaths_us_df` : dataframe，儲存從 2020/1/22 到現在美國死亡人數

{% include notebooks/COVID-19-current-situation-on-December/2.html %}

### 資料前處理

變更日期的格式，由 mm/dd/yy 改成 yy-mm-dd

{% include notebooks/COVID-19-current-situation-on-December/3.html %}

將所有日期合併到同一欄位 (Date) ，該日期的累積人數合併到另一欄位 (Confirmed, Deaths, Recovered)

{% include notebooks/COVID-19-current-situation-on-December/4.html %}

將所有日期合併到同一欄位 (Date) ，該日期的累積人數合併到另一欄位 (Confirmed, Deaths, Recovered)

{% include notebooks/COVID-19-current-situation-on-December/5.html %}

製作訓練用的美國的資料集

{% include notebooks/COVID-19-current-situation-on-December/6.html %}

## 綜觀全球疫情走勢

觀察 `ww_df` 的各個屬性

{% include notebooks/COVID-19-current-situation-on-December/7.min.html %}

把 `ww_df` 的 **confirmed fatalities new_case** 三個屬性分別出來，放在 `ww_melt_df`

{% include notebooks/COVID-19-current-situation-on-December/8.min.html %}

### 全球確診/死亡案例 (折線圖)

- 2020/04/02 確診人數突破 1M，且死亡人數為 52K
- 2020/07/28 確診人數突破 10M，且死亡人數為 500K
- 2020/08/13 確診人數突破 20M，且死亡人數為 750K
- 2020/11/08 確診人數突破 50M，且死亡人數為 1.25M

{% include notebooks/COVID-19-current-situation-on-December/9.html %}

### 全球確診/死亡案例 (折線圖) (取對數)

如果某段期間為斜直線，代表該段期間的案例為指數增長  
比較 2020/3 初和 2020/3 底，確診案例成長率的上升速度略為增加

{% include notebooks/COVID-19-current-situation-on-December/10.html %}

### 全球死亡率 (折線圖)

可以明顯看到，死亡率在 2020 年 5 月達到高峰 (7%)，隨後開始下降，可能是因為醫療能量慢慢提升，或是更多潛在患者確診?

{% include notebooks/COVID-19-current-situation-on-December/11.html %}

### 計算全球新增確診案例成長因子 (折線圖)

公式: 每日新增確診案例 / 昨日新增確診案例
- 大於1: 確診案例增加
- 小於1: 確診案例減少

{% include notebooks/COVID-19-current-situation-on-December/12.html %}

## 分析各國確診與死亡人數變化

宣告 `country_df` 條列各國確診與死亡人數，並查看 `country_df` 的各個欄位

{% include notebooks/COVID-19-current-situation-on-December/13.html %}

列出 dataset 的所有國家 (注意，台灣的標籤為 *Taiwan\** )

{% include notebooks/COVID-19-current-situation-on-December/14.html %}

列出所有國家確診案例的數量級

{% include notebooks/COVID-19-current-situation-on-December/15.html %}

以柱狀圖表示

{% include notebooks/COVID-19-current-situation-on-December/16.html %}

### 目前各國確診/死亡案例 (柱狀圖)

將各國的資料根據確診數排序，排除確診人數小於 1000 的國家 (此時台灣確診 911、 死亡 8)，存在 `top_country_df`  
下文確診案例、死亡案例、死亡率也一樣排除確診人數小於 1000 的國家

{% include notebooks/COVID-19-current-situation-on-December/17.html %}

### 目前確診案例前 10 多的國家 (折線圖)

{% include notebooks/COVID-19-current-situation-on-December/18.html %}

### 目前死亡案例前 10 多的國家 (折線圖)

{% include notebooks/COVID-19-current-situation-on-December/19.html %}

### 目前死亡率最高的前 30 個國家 (柱狀圖)

{% include notebooks/COVID-19-current-situation-on-December/20.html %}

### 目前死亡率最低的前 30 個國家 (柱狀圖)

{% include notebooks/COVID-19-current-situation-on-December/21.html %}

宣告 `all_country_df` 紀錄每個國家最近一天的疫情資訊，並新增以下欄位:
- 計算各國確診數的對數，記為 `confirmed_log1p`
- 計算各國確診數的對數，記為 `fatalities_log1p`
- 計算各國死亡率，記為 `mortality_rate`

{% include notebooks/COVID-19-current-situation-on-December/22.html %}

### 各國確診案例比較 (動態全球地圖) (以顏色區分等級)

顏色越深代表確診案例越多；反之則越少

{% include notebooks/COVID-19-current-situation-on-December/23.html %}

### 各國死亡案例比較 (動態全球地圖) (以顏色區分等級)

顏色越深代表死亡數越多；反之則越少

{% include notebooks/COVID-19-current-situation-on-December/24.html %}

### 各國死亡率比較 (動態全球地圖) (以顏色區分等級)

顏色越深代表死亡率越高；反之則越低

{% include notebooks/COVID-19-current-situation-on-December/25.html %}

此時文中提到歐美澳的死亡率特別高，並提到一項假設是卡介苗在這些國家的接種率很高，也許跟卡介苗注射有關。不過考慮到這篇文章大概在 2020 年 5 到 6 月就有了，那時歐美的死亡率可能真的比較高，現在 (2021 年 2 月) 來看，歐美其實死亡率並沒有比較高。

### 死亡數演變 (折線圖) (取log) (超過 10 人死亡就開始記錄)

可以由這張圖比較各國的防疫能力 (醫療資源較充足、較早開始處理疫情的國家，曲線上升幅度會較緩慢)

{% include notebooks/COVID-19-current-situation-on-December/26.html %}

### 確診人數前十國每日新增確診案例 (折線圖)

{% include notebooks/COVID-19-current-situation-on-December/27.html %}

### 各國疫情發展情況: 確診案例 (點狀動態圖)

{% include notebooks/COVID-19-current-situation-on-December/28.html %}

由於所需的數據大於 512 KB ，超過 chart-stdio 的上限，這裡僅放截圖:

![](https://raw.githubusercontent.com/blueskyson/image-host/master/world-covid19-progress.png)

## 各國省分詳細資訊

提供詳細省分詳細資訊的僅有 8 個國家:  
Australia, Canada, China, Denmark, France, Netherlands, US, UK

{% include notebooks/COVID-19-current-situation-on-December/29.html %}

## 分析美國疫情

因為需要額外登入到 kaggle 下載資料，我就不搬運了，有興趣的可以看[這裡](https://www.kaggle.com/corochann/covid-19-current-situation-on-december/comments#Zoom-up-to-US:-what-is-happening-in-US-now??)

## 分析歐洲疫情

{% include notebooks/COVID-19-current-situation-on-December/30.html %}

### 歐洲國家確診案例 (折線圖)

{% include notebooks/COVID-19-current-situation-on-December/31.html %}

### 歐洲國家死亡案例 (折線圖)

{% include notebooks/COVID-19-current-situation-on-December/32.html %}

### 歐洲每日新增確診案例 (折線圖)

{% include notebooks/COVID-19-current-situation-on-December/33.html %}

## 分析亞洲疫情

{% include notebooks/COVID-19-current-situation-on-December/34.html %}

### 亞洲國家每日新增確診案例 (折線圖)

{% include notebooks/COVID-19-current-situation-on-December/35.html %}

## 哪些國家已經脫離疫情高峰

以下地圖顯示，黃色國家的比例很高，代表確診人數快速上升，處於疫情高峰；藍色和紫色國家的比率較低，代表從高峰期開始下降。

我們可以發現歐美似乎有漸漸脫離高峰期，東南亞正在疫情高峰，德國及澳洲似乎已將疫情控制

{% include notebooks/COVID-19-current-situation-on-December/36.html %}

## 全球疫情何時趨緩

利用 sigmoid 函數建立一個模型協助我們預測各國的疫情何時趨緩 (確診人數不再暴增)

待續...