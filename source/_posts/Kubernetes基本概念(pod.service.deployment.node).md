---
title: Kubernetes基本概念(pod.service.deployment.node)
date: 2020-06-02 18:23:16
tags:
    - k8s
    - 学习笔记
description: k8s涉及到多个层级的概念，对其中的pod、service、deployment、node四个概念进行阐述和关系理顺
---
# pod
pod是Kubernetes抽象出来的最小调度模块，表示一组一个或多个应用程序容器（如 Docker 或 rkt ），以及这些容器的一些共享资源。包括:
- 储存空间
- 网络，以pod为单位，每个pod在集群中有享有一个ip，pod内应用共享这个ip。
- 有关每个容器如何运行的信息，例如容器映像版本或要使用的特定端口。在同一个pod中的镜像可以共享这些信息。
![pod](pod.svg)

# service
尽管每个pod都有一个唯一的IP地址，但是如果没有service，这些IP不会向外暴露。
service将一组相关pod组合成为服务并向外暴露，其允许pod在Kubernetes中进行死亡和复制等生命周期而又不影响应用程序。
![service](service.svg)
service暴露方式有以下几种
- ClusterIP（默认）:在群集的内部IP上公开服务，这种类型使得只能从群集内访问服务。
- NodePort:使用NAT在群集中每个选定节点的相同端口上公开服务，可从群集外部通过<NodeIP>:<NodePort>访问，ClusterIP策略的超集。
- LoadBalancer:在当前集群中创建一个外部负载均衡器，为这个负载均衡器分配一个固定的外部IP，通过这个负载均衡器访问内部服务，NodePort策略的超集。
- ExternalName:通过配置externalName暴露当前服务，通过CNAME配置访问当前服务，无需代理。此类型需要v1.7或更高版本kube-dns。
 
# deployment
在Kubernetes集群中部署容器化应用程序需要创建Kubernetes Deployment配置。
Deployment指挥Kubernetes 如何创建和更新应用程序的实例。
创建Deployment后，Kubernetes master将应用程序实例调度到集群中的各个节点上。
![deployment](deploy.svg)
也就是说，deployment是在master节点上的配置信息

# node
工作节点，Kubernetes中的参与计算的机器，可以是虚拟机或物理计算机，具体取决于集群。
![node](node.svg)
工作节点可以有多个 pod ，Kubernetes master节点会自动根据各个node上的可用资源在群集中调度pod。

# 总结
1. service>pod>容器
    - 同一个pod可以有多个或者单个容器组成最小调度单位，单个pod中的容器共享资源且联系十分紧密，可以看作单个容器无法单独工作。
    - service聚合了多个pod，组合成一个服务，同时提供像集群外暴露端口的能力。
2. node
    - node本身与service、pod、容器并不是同一层面的概念，其代表了集群中具体的物理机(也有可能是虚拟机)。
3. deployment
    - master节点中对程序调度的相关配置。