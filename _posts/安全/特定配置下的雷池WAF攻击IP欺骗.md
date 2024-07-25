---
title: 特定配置下的雷池WAF攻击IP欺骗
date: 2024-07-25 12:52:58
author: medasz
toc: "true"
tags: 
categories: 安全
---
> 最近用到了雷池WAF，感觉很厉害，也非常的流行。
> 
> 由于本地是通过nginx做了反向代理，所以没法通过`从网络连接中获取`的方式直接获取攻击IP，需要通过请求头中加代理header头获取。

![](/images/1.png)
>这种情况下，就可以通过在请求头中自定义添加header头部：`X-Forwarded-For: 127.0.0.1`达到攻击IP的欺骗效果。

例如：
![](/images/Pasted_image_20240725235355.png)
## 防护方式：
- 自定义一个不常用的header存储代理过程中的IP地址，无法让攻击者进行猜解的目的。