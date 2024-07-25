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
> 由于本地是通过nginx做了反向代理，所以没法通过`从网络连接中获取`的方式直接获取攻击IP，需要通过请求头中加代理header头获取

![[Pasted image 20240725130159.png]]