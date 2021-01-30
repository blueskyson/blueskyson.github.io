---
title: "COVID-19: current situation on December (從 Kaggle Notebook 翻譯)"
subtitle: ""
excerpt: "COVID-19 python data science"
layout: post
author: "blueskyson"
header-style: text
mathjax: true
tags:
  - data science
  - python
---

來源: [https://www.kaggle.com/corochann/covid-19-current-situation-on-december/notebook](https://www.kaggle.com/corochann/covid-19-current-situation-on-december/notebook)

此筆記的資料集來源: [Johns Hopkins University github repository](https://github.com/CSSEGISandData/COVID-19)

主要重點:

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
    - 額外把台灣加在裡面
    - 以 `顏色指標亞洲動態地圖`、`顏色指標折線圖`、`顏色指標熱圖` 分析
7. [**哪些國家目前沒有新增案例**](#哪些國家目前沒有新增案例)
    - 以 `表格`、`柱狀圖`、`顏色指標全球動態地圖`、`顏色指標熱圖` 分析
8. [**全球疫情何時趨緩**](#全球疫情何時趨緩)
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

待續...