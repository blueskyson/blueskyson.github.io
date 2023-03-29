---
title: "在筆電執行 Alpaca LoRA"
subtitle: ""
excerpt: "alpaca lora llama llama.cpp"
layout: post
author: "blueskyson"
header-style: text
tags:
  - others
---

[Alpaca-LoRA](https://github.com/tloen/alpaca-lora) 是一個開源專案，它使用 Low-Rank Adaptation (LoRA) 技術重現史丹佛 Alpaca，提供了一個與 [text-davinci-003](https://platform.openai.com/docs/models/gpt-3-5) 相似的 Instruct 模型，他最小可小到在 Raspberry Pi 上運行（用於研究），也可以擴展到 13b、30b 和 65b 模型。聽到 Alpaca-LoRA 可以把模型縮小到在不依賴 GPU 的情況下在個人電腦上運行讓我覺得很驚豔，結果自己嘗試了一下在自己筆電運行，並寫下這篇筆記留念一下。整個過程在 i7-10750H CPU，32 GB RAM 上進行，作業系統是Ubuntu 22.04。

## Alpaca-LoRA

原始碼: https://github.com/tloen/alpaca-lora

執行程式

![](https://raw.githubusercontent.com/blueskyson/image-host/master/2023/alpaca-1.png)

資源使用量 (完全吃滿 32 GB RAM)

![](https://raw.githubusercontent.com/blueskyson/image-host/master/2023/alpaca-2.jpg)

輸入和輸出的截圖，以下每張傑圖的輸出都需要耗時 330 到 420 秒。
問它一個範例問題：

![](https://raw.githubusercontent.com/blueskyson/image-host/master/2023/alpaca-3.png)

讓它寫給我一個 BMI 計算機：
BMI 程式是正確的，它還提供了參數註解。

![](https://raw.githubusercontent.com/blueskyson/image-host/master/2023/alpaca-4.png)

讓它寫給我一個河內塔程式：
程式看起來是錯的。

![](https://raw.githubusercontent.com/blueskyson/image-host/master/2023/alpaca-5.png)

用簡體中文問它「什麼是機器學習？」並讓它用中文回答：
它能讀懂中文，但只能用英文回答。

![](https://raw.githubusercontent.com/blueskyson/image-host/master/2023/alpaca-6.png)

## llama.cpp

原始碼: https://github.com/ggerganov/llama.cpp

一個叫 ggerganov 的 github 帳號用 C 寫了一個用於機器學習的 tensor 函式庫 ggml。然後他把 ggml 移植到 LLaMA 模型上，然後把 16 位的 float 改成更小的數值型態，用 AVX2 或者 NEON 在 CPU 上加速。詳細資訊請參閱他的[評論](https://github.com/ggerganov/llama.cpp/issues/33#issuecomment-1465108022)。

最小的模型 alpaca-lora-7B-ggml 大小僅僅 4GB，可以在樹莓派上運行，見此[影片](https://www.youtube.com/watch?v=eZQTYTst53o)。我原本也想試試這個 4 GB 的模型，但最近這個專案有 breaking change，不確定這個模型現在是否有效，所以我嘗試了 alpaca-lora-30B-ggml，他需要 20 GB 的硬碟空間。它仍然執行得很慢，並且在互動式終端機中似乎有一個小 bug，希望以後可以改進，讓這個模型能更堪用。

![](https://raw.githubusercontent.com/blueskyson/image-host/master/2023/alpaca-7.jpg)

我錄下了影片，它真的執行的很慢：[https://www.youtube.com/watch?v=RgSAe8tDfew](https://www.youtube.com/watch?v=RgSAe8tDfew)
