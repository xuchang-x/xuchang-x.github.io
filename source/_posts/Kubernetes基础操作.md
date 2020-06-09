---
title: Kubernetes基础操作
date: 2020-06-02 15:03:32
tags:
    - k8s
    - 学习笔记
description: kubenetes.io官方tutorial基础操作学习笔记
---
# 官方文档地址
[k8s官网](https://kubernetes.io/):https://kubernetes.io/
[指引](https://kubernetes.io/docs/tutorials/):https://kubernetes.io/docs/tutorials/

# 创建集群
这里创建集群用的是k8s官网的在线交互系统，其中的两个命令
- `minikube version`    查看minikube版本
- `minikube start`  部署应用

平常应该不太行，看起来minikube是这个在线测试系统配置的命令

# 部署应用(部署到pod)
- `kubectl version` kubectl版本
- `kubectl get nodes`   获取集群中节点信息
- `kubectl create deployment kubernetes-bootcamp --image=gcr.io/google-samples/kubernetes-bootcamp:v1`
    - `kubectl create deployment`
    - 应用名
    - 使用的镜像
- `kubectl get deployments` 获取部署的应用信息

- `kubectl proxy` 代理
    - 代理后可以通过指定端口`curl http://localhost:8001/version`访问应用

# 获取应用信息
- `kubectl get` 列出资源
- `kubectl describe`    显示有关资源的详细信息
- `kubectl logs`    打印pod和其中容器的日志
- `kubectl exec`    在pod中的容器上执行命令

上述四个命令并非完整命令，比如需要获得pod相关信息则需要将命令细化为如下所示用法：

- `kubectl get pods`    获取pod列表
- `kubectl describe pods`    获取pod描述信息
    ![kubectl describe pods](kubectl%20describe%20pods.jpg)
    
## 获取pod运行结果
1. <code>export POD_NAME=$(kubectl get pods -o go-template --template '&#123;&#123;range .items&#125;&#125;&#123;&#123;.metadata.name&#125;&#125;&#123;&#123;"\n"}}&#123;&#123;end&#125;&#125;')`</code>
获得pod名字并且输出成变量
2. `curl http://localhost:8001/api/v1/namespaces/default/pods/$POD_NAME/proxy/`输出pod内容
3. `kubectl logs $POD_NAME` 获取pod里的运行日志

    *TODO:*
    1. 这里应该不是固定url就能拿到内容，如果pod里同时暴露了两个url显然是不能通过这一个url同事获得的。
    2. proxy 只用`kubectl proxy`就代理到8001端口了，后续继续细化一下如何把pod的指定端口暴露到外部的指定端口。

## 在指定pod中运行命令
1. `kubectl exec $POD_NAME env` 在指定pod中运行env命令，效果如下
    ![kubectl exec $POD_NAME env](kubectl%20exec%20$POD_NAME%20env.jpg)
    *TODO:* 多个pod同时运行命令如何操作，pod名之间用`,`分隔么
2. `kubectl exec -ti $POD_NAME bash`在pod中开启一个bash，这里和docker一样，通过`-it`参数使用交互式bash
3. `exit`   退出container

# 发布应用
1. `kubectl expose deployment/kubernetes-bootcamp --type="NodePort" --port 8080`    暴露service
2. `kubectl label pod $POD_NAME app=v1`通过label命令为指定pod(也有可能是其他粒度)贴标签
3. `kubectl get pods -l app=v1`即可通过label获取相关pod;`kubectl describe pods -l app=v1`可获得相关描述
    也就是可以用`-l app=v1`替代`$POD_NAME`
4. `kubectl delete service -l run=kubernetes-bootcamp`  删除应用

# 扩容缩容
扩容缩容是通过改变 Deployment 中的副本数量来实现的
- `kubectl get deployments` 获取deployment信息
    ![kubectl get deployments](kubectl%20get%20deployments.jpg)
    - NAME:集群中deployments的名称信息
    - READY: 已部署副本/目标副本数比例
    - UP-TO-DATE: 已完成更新达到目标副本要求的副本数
    - AVAILABLE: 全部可用副本数
    - AGE:应用运行时长
- `kubectl get rs`  查看应用创建的副本集
    ![副本集 扩容前](副本集%20扩容前.jpg)
    注意这里是副本集，不是每个副本都有单独的名字。
    扩容后如下
    ![副本集 扩容后](副本集%20扩容后.jpg)
    副本集命名规则`[DEPLOYMENT-NAME]-[RANDOM-STRING]`
- `kubectl scale deployments/kubernetes-bootcamp --replicas=4`  修改副本数，从而实现扩容缩容
此时通过`kubectl get pods -o wide`可以看到pod数已经扩容到4。当有多个实例运行时，打到服务上的请求会进行负载均衡。

# 更新应用
Kubernetes通过滚动更新（Rolling Updates）使用新的实例逐步更新Pod，更新过程中零停机。过程如下:
![滚动更新过程1](滚动更新过程1.svg)
![滚动更新过程2](滚动更新过程2.svg)
![滚动更新过程3](滚动更新过程3.svg)
![滚动更新过程4](滚动更新过程4.svg)
- `kubectl set image deployments/kubernetes-bootcamp kubernetes-bootcamp=jocatalin/kubernetes-bootcamp:v2`更新镜像
    - `kubectl set image`
    - deployment名称
    - 容器名
    - 新镜像
- `kubectl rollout status deployments/kubernetes-bootcamp`  查看更新状态（是否更新完成）

## 发布错误回滚
- `kubectl set image deployments/kubernetes-bootcamp kubernetes-bootcamp=gcr.io/google-samples/kubernetes-bootcamp:v10` 发布了一个错误的镜像。

这个时候我们通过`kubectl get deployments`显示状态有三个pod正在运行，两个pod是最新镜像
![发布失败](发布失败.jpg)
这时候通过`kubectl get pods`和`kubectl describe pods`看到也没有镜像不对，k8s在检测到错误后保留了剩余节点继续提供服务。
- `kubectl rollout undo deployments/kubernetes-bootcamp`    回滚发布
    - `kubectl rollout undo`
    - deployment名称