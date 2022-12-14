---
title: "OpenCV 圖片混合"
subtitle: ""
excerpt: "opencv addWeighted 影像混合"
layout: post
author: "blueskyson"
header-style: text
mathjax: true
tags:
  - opencv
  - python
---

OpenCV 圖片混合的函式為：

```python
cv2.addWeighted(src1, alpha, src2, beta, gamma, dst=..., dtype=...) -> dst
```

必要參數

- **src1:** 第一張圖片的 numpy array。
- **alpha:** 第一張圖片的權重。
- **src2:** 第二張圖片的 numpy array。
- **beta:** 第二張圖片的權重。
- **gamma:** 圖片混合後添加的常數。

非必要參數

- **dst:** 輸出的圖片的 numpy array。
- **dtype:** 輸出影像的位元深度，預設為 `src1.depth()`。

混合的公式可以表示如下：

$\texttt{dst} (I)= \texttt{saturate} ( \texttt{src1} (I)* \texttt{alpha} + \texttt{src2} (I)* \texttt{beta} + \texttt{gamma} )$

## 範例

以下範例設定 alpha + beta = 1.0，並且 gamma = 0.0。此範例也放在我的 [github repo](https://github.com/blueskyson/image-blending-opencv)。執行腳本後按下 `ESC` 退出腳本。

```python
import cv2


def main():
    img1 = cv2.imread("Dog_Strong.jpg")
    img2 = cv2.imread("Dog_Weak.jpg")
    window_name = "Blending"

    def change_trackbar(val):
        alpha = float(val / 255)
        blend_img = cv2.addWeighted(img2, alpha, img1, 1.0 - alpha, 0.0)
        cv2.imshow(window_name, blend_img)
    
    cv2.namedWindow(window_name)
    cv2.createTrackbar("trackbar", window_name, 0, 255, change_trackbar)
    cv2.imshow(window_name, img1)
    while True:
        if cv2.waitKey(100) == 27:  # ESC
            cv2.destroyWindow(window_name)
            break


if __name__ == "__main__":
    main()
```

Demo:

![](https://raw.githubusercontent.com/blueskyson/image-blending-opencv/main/demo.gif)