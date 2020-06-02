---
title: k8s笔记
tags:
    - k8s
description: 仅用来暂时记录一些小问题，后边会整理成文
---
- `kubectl run` 部署应用
    - ```shell script
      kubectl run kubernetes-bootcamp \
      --image=docker.io/jocatalin/kubernetes-bootcamp:v1
      --port=8080
 
      ```
      image:指定镜像
      
      port: 指定对外服务接口
- `kubectrl get pods`   查看当前pod

- `kubectl cluster-info`    查看集群信息