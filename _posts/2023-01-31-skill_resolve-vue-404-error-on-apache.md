---
title: vue项目部署到Apache后子界面刷新出现404的解决办法
date: 2023-01-31 00:26:00 +0800
categories: [技巧]
tags: [vue]
pin: false
---

vue项目部署到Apache后，在子界面刷新出现404，以下是解决办法。在项目根目录添加Apache重写规则文件 `.htaccess`，加入以下代码

```shell
<IfModule mod_rewrite.c>
  RewriteEngine On
  RewriteBase /
  RewriteRule ^index\.html$ - [L]
  RewriteCond %{REQUEST_FILENAME} !-f
  RewriteCond %{REQUEST_FILENAME} !-d
  RewriteRule . /index.html [L]
</IfModule>
```

重启Apache服务器即可解决
