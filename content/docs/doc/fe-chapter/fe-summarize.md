---
title: 掘金技术总结
description: 掘金技术总结
toc: true
authors:
tags:
categories:
series:
date: '2022-07-31'
lastmod: '2022-07-31'
draft: false
---

稀土掘金大会技术文章总结

<!--more-->

## Intel 云计算相关

isa: 指令集
XPU: 其实处理器的架构并没有改变，它们仍然遵守过去 40 年来一直遵循的规则。变的是芯片的构造方式，它们现在包含大量异构处理器，这些芯片根据各自的任务，对内存和通信进行优化。每个芯片都对处理器性能、优化目标、所需的数据吞吐量以及数据流做出了不同的选择。

## goole 云原生
                                           Vertex AI
Firebase        Firestore       BigQuery   Remote PDF
Cloud Fucntions                 cloud Run

可伸缩性 / 可观察性/ 供应链安全

web3.0

### IEEE Fellow 智能人机交互

略

### 尤雨溪

1. 开发范式 / 底层框架
    1.1 React Hooks （状态，副作用，状态更新）
      Svelte / Solid
      useEvent RFC (TODO)
      基于依赖追踪的范式（痛点：react useEffect 需要声明依赖）（过期的闭包问题）

    1.2 基于编译时的优化方案：
    - Vue Reactivity transform
    
    1.3 基于编译时的运行优化
    - Vue Vapor Mode (一次性生产静态节点 => 生产动态节点)

2. 工具链
  2.1 工具链的抽象层次
    vite(API 层次)

3. 上层框架 （Next, Nuxt, Remix）
  3.1 数据流的打通
  3.2 类型的前后打通
  3.3 全栈的代价
  - hydrate
  - server only
  3.4 探索的方向


### 技术人如何做产品（玉伯）（？？？）
  完全没有干货....

### 码上掘金（月影）
  codepen, codesandbox, stackblitz
  黑科技：
   - esModule
   - vue, react 等支持
   - JCode Tools
   - webSlides (ppt)
   - 牛客面试题
  

### 云原生

1. CI / CD 云原生交付链 (cncf社区)
  Branch - Validate - Review - Merge - Deploy
  - 推荐工具
    * Istio 中的服务模型，它既可以支持 Kubernetes 中的工作负载，又可以支持虚拟机。

    * 云原生打包工具：Buildpacks

    * Skaffold （CodeChanging - Package and Build Image - push Container Image - update K9s Config - Apply K8s Config - Verfiy and Test）

    * Kustomize (hydrate / patch)
    helm 的升级

    * anthos config management 
    Flux, Argo CD (主要职责都是监听Git Repository源中的应用编排变化，并与当前环境中应用运行状态进行对比，自动化同步拉取应用变更并部署到进群中。)

    * Jenkisn & Gitlab Runner / Cloud Build depoy

### serverlesss (无状态应该，不依赖缓存，事件触发架构体系)
  1. 介绍
    一个函数算子就是一个k8s的pod
    冷启动问题（资源不使用就先关闭，所以存在冷启动问题）
    dapr (布式应用运行)
     
  2. 高并发
    单实例多并发
    离线场景： 绕过getway
    外部流量：入流量，出流量，每个容器内部 Mesh 代理

  3. 微服务，多协议(http,grpc,thrift)

    http协议优势：无须解析body,识别不同租户的请求
    grpc (http2)
  
  4. 端云联调/前后端全链路灰度/可观测（tracing, metrics,logging）
   

### 大前端工程实践和性能优化

  - 内存性能准出：
    无法判断webview的内存开销 ，判断占比的类比
    Chrome DevTools Protocal 性能检测工具

  - 微前端 ci / cd 的复杂度

  - ide 相关

  - 依赖预构建 （第三方包预构建）

  - 接口文档 会 生成 apiService (这个结合 gitlab 的 snippets 是不是很有价值)
    结合 swagger 的 JSON 文件

  - 性能方面： 业务指标 技术指标

    https://github.com/microsoft/applicationinsights-js 看下
    Time to Airbnb Interactive
    Page Performance Score

### 低代码平台

 - 调试工具
   * 组件审查 
    数据源面板 / schema面板

- 低代码拓展工具
  * 拓展调试能力

  协议栈： 物料描述协议/页面搭建协议
  低代码引擎： 入料 编排 渲染 出码  （LowCodeEngine)[https://zhuanlan.zhihu.com/p/487477918]
  引擎生态：插件 物料 设置器 工具链
  低代码平台

  低代码调试协议

  域 - 容器 - 单个组件结构 - runtime - overlay

- 多人协作

  CRDT, yjs(多人协作方案)
  automerge

- 可视化运营活动编辑设计
 
  tmagic editor

  runtime , DSL

  NODE 包起来，created 作为 二次开发的 procode