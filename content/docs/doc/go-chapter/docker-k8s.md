---
title: 容器化相关
description: 容器化相关
toc: true
authors:
tags:
categories:
series:
date: '2020-10-16'
lastmod: '2020-10-16'
draft: false
---

容器化相关

<!--more-->

### 参考文献
[安装k8s](https://www.jianshu.com/p/a6abdc6f76e1)

### ------------------------------------ 命令:context 相关 ------------------------------------

##### 集群用户和上下文：

kubectl config --kubeconfig=minikueb view

kubectl config use-context xxxx // 切换多集群中的一个

### 命令 namespace 相关:

kubectl get namespace



kubectl cluster-info
kubectl get nodes
kubectl describe node

kubectl create namespace adhoc  //namespace 相当于作用域
kubectl get namespaces

kubectl get pods --namespace kube-system 看看状态是否running
### k8s:

管理多个主机的容器化的应用


##### ------------------------------------ 概览 ------------------------------------

所有的容器都在pod中：
同一个pod中的容器会部署在同一个物理机器上，并且能够共享资源
* 服务发现和负载均衡
* 存储编排
* 自动化部署和回滚
* 自我修复
* 密钥与配置管理

##### 组件

* 控制面板组件（对集群做出全局决策）
  * kube-apiserver
  * etcd 
  * kube-scheduler
  * kube-controller-manager
    控制器（内置）：Deployment 控制器和 Job 控制器是 Kubernetes 内置控制器的典型例子

* Node组件（节点组件）
  * kubelet PodSpecs  Node 端 保证 *容器* 运行 在 *pod* 中
    （kubelet 不会管理不是由kubernetes创建的容器）
  * kube-proxy 
    运行在Node上的网络代理，实现kubernetes服务(service)的一部分
  * Container Runtime 容器运行时
    Kubernetes 支持容器运行环境: docker

  > 容器 - pod - 节点 - 集群
    注册：节点上的 kubelet 自注册，json对象
    ```json
    {
        "kind": "Node",
        "apiVersion": "v1",
        "metadata": {
            "name": "10.240.79.157",
            "labels": {
            "name": "my-first-k8s-node"
            }
        }
    }
    ```
    ``` shell
    #常用命令
    kubectl describe node
    ```
    

* 插件
  > 插件使用 Kubernetes 资源 实现集群功能，以为这些插件提供集群级别功能。插件中命名空间域的资源属于 *kube-system* 命名空间
  * DNS
  * Dashboa
  * 容器资源监控
  * 集群层面日志

* 理解Kubernetes 对象

  实体 表现 集群
  1. 哪些容器化应用在运行
  2. 可以被应用使用的资源
  3. 策略：重启，升级，容错   

  对象约束 状态 Spec Status
  Spec 期望值
  Status 


  ```shell
  kubectl apply -f
  kubectl run xxx #创建Deployment运行容器
  ```



 #### 如何理解一个工程:
* Job：
  .spec.template 为必填字端

### ------------------------------------ 命令相关 ------------------------------------



### kubelet

kubelete 作为节点代理，每个节点都有kubelete金川，处理master下发的任务（Podspec:yaml => pod）

容器检测 && 容器健康管理

### kukbeadm 

用来部署 kubernetes 集群
组件间访问授权，健康检查


### 备注：

Pod 
可以容纳多个容器
相同的pod中的任何容器共享相同名称空间和本地网络

pod 由 deployment（部署） 来进行管理
：
声明一个pd应该同时运行多个副本
不必手动处理pod，只需声明系统的期望状态

### 尝试使用kubernete 对象

``` shell

$ kubectl config use-context docker-desktop



$ goctl kube deploy -name nodejs -namespace keepfe -image ccr.ccs.tencentyun.com/keep/nodejs-build -o nodejs.yaml -port 6379
$ kubectl create namespace keepfe
$ kubectl apply -f nodejs.yaml
$ kubectl get all -n nodejs
$ kubectl run -i --tty --rm cli --image=nodejs -n keepfe -- sh
$ kubectl get pods -n keepfe
$ kubectl get pods -n keepfe -o yaml 
$ kubectl exec -it cli -n keepfe /bin/sh  #执行pod里面的 容器
$ kubectl exec -ti <your-pod-name>  -n <your-namespace>  -- /bin/sh
$ kubectl delete pod nodejs-577ccc9697-869j4 -n keepfe
$ kubectl get deployment -n keepfe
$ kubectl delete deployment nodejs -n keepfe


# 查看日志
$ kubectl -n default logs -f goweb
```

minikube
[参照链接](https://segmentfault.com/a/1190000018607114)
```shell
$ minikube ssh
  #docker ps | awk '{print $2F}'
$ kubectl config view
$ kubectl get node -o wide
  # 相当于用yaml 创建 pod
$ kubectl run goweb --image=hello  --port=8888 --replicas=3
$ kubectl create deployment goweb --image=192.168.3.85:5000/netkiller/welcome:latest
$ kubectl expose deployment goweb --port=8888 --target-port=8888 --type=NodePort
$ minikube service welcome --url
$ curl http://192.168.64.5:30257/from/you

# 报错指南
进入node节点去看
```

集群 节点 pod 服务



# 宿主本地源

``` json
{
  "insecure-registries" : [
    "10.21.71.237:5000"
  ],
  "debug" : true
}
```

```shell
$ docker pull registry
$ docker run -d -p 5000:5000 -v $(pwd):/var/lib/registry --restart always --name registry registry:2
$ docker tag hello:v1 10.2.17.5:5000/hello-local

```

```
minikube delete && minikube start --cpus=2 --memory=4096 --disk-size=10g \
  --driver=hyperkit \
  # --docker-env http_proxy=10.2.17.5:1087 \
  # --docker-env https_proxy=10.2.17.5:1087 \
  # --docker-env no_proxy=192.168.99.0/24 \
  --registry-mirror=https://registry.docker-cn.com \
  --insecure-registry=10.2.17.5:5000 \
  --service-cluster-ip-range='10.10.0.0/24'
```

#https://zhuanlan.zhihu.com/p/261722859



####  ------------------------------------ helm 相关：------------------------------------

- helm 相当于 k8s 的包管理工具：

- helm repo add stable xxxxxxx     ## 添加源

- helm install stable/redis --generate-name

- helm install -f values.yaml bitnami/wordpress --generate-name

- helm template sentry/sentry  > resource.yaml

- kubectl run --namespace infra redis-1619522403-client --rm --tty -i --restart='Never' --env REDIS_PASSWORD=AAerZRT8gv%  --image docker.io/bitnami/redis:5.0.7-debian-10-r32 -- bash

### ---------------------------- k8s king 类型 ----------------------------------------

  * config 配置文件
  * deployment 部署文件
  * service 服务
  * Ingress
  * job 任务
k8s 配置文件事例
```yaml
apiVersion: v1 # 指定api版本，kubectl apiversion
kind: pod
metadata: #资源元属性
  name: test-pod #资源的名字
  labels: #设定资源的标签
  annotations：自定义注解列表
specs: #指定该资源的内容
  replicas： 3 #定义有多少个pod
  restartPolicy: # 退出重启
  nodeSelector：#节点选择(节点归类，ssd)
  containers： #容器
  - name
volumes：#定义一组挂载设备
- name: 设备名称

```

 kubectl get service keep-admin-webapp -o yaml

