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

**ERROR: Could not install packages due to an OSError: [Errno 2]**

```
> pip3 install requests
ERROR: Could not install packages due to an OSError: [Errno 2] No such file or directory: 'c:\\users\\lin\\anaconda3\\envs\\env1\\lib\\site-packages\\chardet-3.0.4.dist-info\\METADATA'
```
-- 解法: 刪除 C:\Users\Lin\anaconda3\envs\env1\Lib\site-packages\chardet-3.0.4.dist-info 然後重新安裝 requests