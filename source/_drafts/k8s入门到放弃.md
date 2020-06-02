---
title: k8s入门到放弃
date: 2020-05-29 17:34:00
tags:
    - k8s
    - 阅读笔记
description: 2017年10月docker宣布原声支持kubernetes，主流的AWS、腾讯云、阿里云等公有云提供的也都是基于k8s的服务。和其他技术的学习一样，先来一次从入门到放弃。
---
# 文档
## 官方文档
[k8s官网](https://kubernetes.io/)、[指引](https://kubernetes.io/docs/tutorials/)

## 中文社区
[社区](https://www.kubernetes.org.cn/)、[中文文档](https://www.kubernetes.org.cn/docs0)

Deployment：一个应用
Pod 容器的集合，紧密相关的一组容器在统一个pod中国，共享ip和port空间，也就是说在同一个network namespace中。
pod是k8s调度的最小粒度