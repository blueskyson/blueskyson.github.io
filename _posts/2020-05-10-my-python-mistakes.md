---
title: "撰寫 python 時遇到的錯誤"
subtitle: ""
layout: post
author: "blueskyson"
header-style: text
tags:
  - python
---

**使用 jupyter notebook 時出現 "ImportError: No module named seaborn" 訊息**

-- 解法: 試試看使用 pip3 安裝 seaborn ，用 pip 或 conda 安裝的套件目錄可能不在 python3 的目錄，導致 jupyter 沒有抓到該套件
