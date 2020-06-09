---
title: k8s命令
date: 2020-06-04 10:49:19
tags: 
    - k8s
---

重要的几个组件`kubelet`、`kubeadm`、`kubectl`
- kubelet直接运行在节点上，负责启动pod和容器
- kubeadm用于初始化cluster
- kubectl是Kubenetes命令行工具，用于管理应用

# 创建集群
- `kubeadm init --apiserver-advertise-address 192.168.56.105 --pod-network-cidr=10.244.0.0/16`
    - apiserver-advertise-address:指明master使用哪个interface与其他节点通信。
    如果master有多个interface建议明确指定。不指定默认选择有网关的interface
    - 在这一步中会生成token
    - 会生成KubeConfig文件，kubelet会使用这个文件与master通信
- `kubeadm join --token d30a01.13653e584ccc190 192.168.56.105;6443`
    - TODO:这里的端口号是固定的么

# 部署应用
- `kubectl run nginx-deployment --image=nginx:1.7.9 --replicas=2`
- `kubectl appliy -f nginx.yml`
- `kubectl create`
- `kubectl replace`
- `kubectl edit`
- `kubectl patch`
- `kubectl delete deployment nginx-deployment`
- `kubectl delete -f nginx.yml`
*补一个yml示例*
- `kubectl taint node k8s-master node-role.kubernetes.io/master-`   将master节点当作node节点，可以调度pod到该节点
- `kubectl taint node k8s-master node-role.kubernetes.io/master="":NoSchedule"`   master节点恢复master only状态
- `kubectl label node k8s-node1 disktype==ssd`
- `kubectl get node --show-labels`
```yaml
spec:
  template:
    spec:
      nodeSelector:
        disktype: ssd
```
- `kubectl label node k8s-node1 disktype-`  删除disktype标签，-就是删除，上边的master也一样
DaemonSet:每个node最多运行一个副本。这里并不是说每个节点只能运行一个应用，可以运行多个应用，
但是每个应用在同一个node上只能运行一个副本
- `kubectl get daemonset --namespace=kube-system`
服务类容器，工作累容器
工作累荣日，一次性任务->JOb
- `kubectl get pod --show-all`  查看所有容器，包括Completed的Pod
- `kubectl log myjob-nfkxx`
#  第六章 SERVICE
- `kubectl api-versions`    查看可用的api-version
    - TODO:哪个是给job哪个是给deployment的，怎么知道匹配关系及其特性
iptables 管理cluster到pod的ip
- `iptables-save`   打印当前节点的iptables规则
- `nslookup xxx`    查看xxx的dns信息
- `kubectl `
- `kubectl `
- `kubectl `
- `kubectl `
- `kubectl `
- `kubectl `