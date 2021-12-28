---
title: "OpenCV 讀檔、寫檔、分離 RGB 通道"
subtitle: ""
excerpt: "opencv imread imwrite split merge BGR"
layout: post
author: "blueskyson"
header-style: text
mathjax: true
tags:
  - opencv
  - python
---

安裝 opencv-python

```non
$ pip install opencv-python
```

## OpenCV 讀檔

將圖片命名為 image.jpg 並放在與下方 python 檔案同一個目錄下。

```python
# showimg.py
import cv2
img = cv2.imread("image.jpg")
cv2.imshow("Image", img)
cv2.waitKey(0)
```

然後執行

```non
$ python showimg.py
```

![](https://raw.githubusercontent.com/blueskyson/image-host/master/opencv/1.png)

按鍵盤任意鍵就可以關閉視窗，千萬不要用滑鼠按右上角紅色叉關閉圖片，程式會進入無窮迴圈。想避免手殘可以改用以下寫法。

```python
# showing.py
import cv2
img = cv2.imread("image.jpg")
while True:
    cv2.imshow("Image",img)
    # 27 is keycode of ESC
    if cv2.waitKey(100) == 27:
        cv2.destroyWindow("Image")
        break
```

這個寫法要按下 `ESC` 才會關閉視窗並結束程式，不會有無窮迴圈。

如果要以灰階形式讀取圖檔，就在 `imread` 的第二個參數填上 `cv2.IMREAD_GRAYSCALE` 即可。

```python
img = cv2.imread("image.jpg", cv2.IMREAD_GRAYSCALE)
```

此外，也可以使用 `cv2.cvtColor` 來把彩色圖像變成灰階。

```python
img = cv2.imread("image.jpg")
gray_img = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)
```

灰階圖像如下

![](https://raw.githubusercontent.com/blueskyson/image-host/master/opencv/2.jpg)

## OpenCV 寫檔

使用 `imwrite` 即可存檔成副檔名對應的圖片格式

```python
# showimg.py
import cv2
img = cv2.imread("image.jpg")
cv2.imwrite('save.jpg', img)
cv2.imwrite('save.png', img)
```

## RGB 通道

OpenCV 讀取的圖片其實是一個多維的 numpy array，如果是彩色影像，則圖片中每個像素由左到右，按照藍、綠、紅的數值排列（BGR channel）。

![](https://raw.githubusercontent.com/blueskyson/image-host/master/opencv/3.png)

在終端機打印 numpy array 的維度看起來如下

```python
img = cv2.imread("image.jpg")
print(img.shape)
```

```non
(387, 620, 3)
```

3 代表 BGR 三個 channel、620 代表圖片的寬度、387 代表圖片的高度，所以這是一張 620x387 的 BGR 圖片，我們再打印這張圖。

```python
print(img)
```

```non
[[[169 159 142]
  [169 159 142]
  [169 159 142]
  ...
  [ 95 114 122]
  [ 98 114 121]
  [ 98 114 121]]

 ...

 [[ 20 130 102]
  [ 29 139 111]
  [ 28 135 108]
  ...
  [230 195 131]
  [231 195 131]
  [232 196 132]]]
```

可以看出三個數值一組代表一個像素，以像素為元素形成二維陣列，每個 BGR 數值都介於 0 到 255。灰階影像就不會以 BGR 三個一組為單位，而是直接以 0 到 255 表示每個像素的灰階程度。

接下來透過 `split` 將 BGR 分為三個獨立的二維陣列，再透過 `merge` 把其他不要顯示的 channel 填上 0 以分離 BGR 三通道。

```python
import cv2
import numpy as np
img = cv2.imread("image.jpg")
zero_channel = np.zeros(img.shape[0:2], dtype = "uint8")

B, G, R = cv2.split(img)
imgB = cv2.merge([B, zero_channel, zero_channel])
imgG = cv2.merge([zero_channel, G, zero_channel])
imgR = cv2.merge([zero_channel, zero_channel, R])

cv2.imshow("B", imgB)
cv2.imshow("G", imgG)
cv2.imshow("R", imgR)
cv2.waitKey(0)
```

![](https://raw.githubusercontent.com/blueskyson/image-host/master/opencv/4.jpg)

如果要用 BGR 三通道合成原圖，則用以下程式碼

```
imgBGR = cv2.merge([B, G, R])
```