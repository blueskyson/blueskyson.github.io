---
title: "node 遇到 ERR_OSSL_EVP_UNSUPPORTED 的解法"
subtitle: ""
excerpt: "node npm vue webpack ERR_OSSL_EVP_UNSUPPORTED"
layout: post
author: "blueskyson"
header-style: text
tags:
  - others
---

執行有點舊的 vue.js 專案時，執行 `npm run serve` 遇到以下錯誤：

```non
{
  opensslErrorStack: [ 'error:03000086:digital envelope routines::initialization error' ],
  library: 'digital envelope routines',
  reason: 'unsupported',
  code: 'ERR_OSSL_EVP_UNSUPPORTED'
}
```

此時設定以下環境變數再重新執行 `npm run serve` 即可

```non
export NODE_OPTIONS=--openssl-legacy-provider
```

詳細發生原因參考 [https://stackoverflow.com/a/69746937/13103152](https://stackoverflow.com/a/69746937/13103152)