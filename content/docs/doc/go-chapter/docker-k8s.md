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

### 命令：
kubectl cluster-info
kubectl get nodes
kubectl describe node

kubectl create namespace adhoc  //namespace 相当于作用域
kubectl get namespaces

kubectl get pods --namespace kube-system 看看状态是否running
### k8s:

管理多个主机的容器化的应用

##### 概览

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





kind:

deployment
node