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
工作类容器，一次性任务->JOb
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
# 第八章 health check
- 滚动更新通过参数`maxSurge`和`maxUnavailable`控制替换副本数量
    - maxSurge 控制滚动更新过程中副本综述超过DESIRED的上线，可以是整数，也可以是百分比，向上取整，默认25%
    - maxUnavailable 控制滚动更新过程中，不可用副本站DESIRED的最大比例，可以是整数，也可以百分比，向下取整。可用副本书至少=10-roundDown(10-25%)
    - maxSurge越大，初始创建新副本数量越多；maxUnavailable越大，初始销毁旧副本数量越大。

# 第九章 Secret & Configmap
## Secret
保存敏感信息
创建四种方式
- `kubectl create secret generic myscret --from-literal=username=admin --from-literal=password=123456`
    每个 --from-literal 对应一个信息条目
- ```shell script
    echo -n admin > ./username
    echo -n 123456 > ./password
  kubectl create secret generic myscret --from-file=./username --from-file=./password
    ```
- ```shell script
    cat << EOF > env.txt
  username=admin
  password=123456
  EOF
    kubectl create secret generic myscret --from-env-file=env.txt
    ```
  
- yml
    ```yaml
    apiVersion:v1
    kind:Secret
    metadata:
      name:mysecret
    data:
      username:YWRtaW4=
      password:MTIzNDU2
    ```
    其中两个值是通过base64编码后的结果
    
## ConfigMap
保存非敏感配置信息，以明文方式存储
创建四种方式
- `kubectl create configmap myconfigmap --from-literal=config1=xxx --from-literal=config2=yyy`
    每个 --from-literal 对应一个信息条目
- ```shell script
    echo -n xxx > ./config1
    echo -n yyy > ./config2
  kubectl create configmap myconfigmap --from-file=./config1 --from-file=./config2
    ```
- ```shell script
    cat << EOF > env.txt
  config1=xxx
  config2=yyy
  EOF
    kubectl create configmap myconfigmap --from-env-file=env.txt
    ```
  
- yml
    ```yaml
    apiVersion:v1
    kind:Secret
    metadata:
      name:mysecret
    data:
      config1:xxx
      config2:yyy
    ```
    
- `kubectl get secret` 查看存在的key
- `kubectl describe secret` 查看相关描述
- `kubectl edit secret mysecret` 查看值
编译反编译
echo -n admin | base64 编译
echo -n YWRtaW4= | base64 --decode 反编译

# 第11章 Helm——k8s包管理器
- helm search 查看可安装的chart
- helm repo list 查看仓库地址
- helm repo add 添加仓库，比如企业私有仓库
- helm install stable/mysql 安装chart
报错 no available release name found是因为Tiller权限不足
添加如下命令添加权限后再次执行即可
```shell script
kubectl create serviceaccount --namespace kube-system tiller
kubectl create clusterrolebinding tiller-cluster-rule --clusterrole=cluster-admin --serviceaccount=kube-system:tiller
kubectl patch deploy --namespace kube-system tiller-deploy -p '{"spec":{"template":{"spec":{"serviceAccount":"tiller"}}}}'
```

