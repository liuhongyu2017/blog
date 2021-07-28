---
title: vue部署到nginx上刷新后404
tags: [nginx, vue]
date: 2021-07-22 09:53:19
categories: vue
---

#  vue部署到nginx上刷新后404

vue路由使用**history 模式**时候，nginx 需要添加 try_files $uri $uri/ /index.html 配。

```nginx
location / {
  try_files $uri $uri/ /index.html;
}
```

