---
title: go-zero 微服务（一） 以及 相关初探
description: 面向 微服务的的一次 越疱代徂 的分享
toc: true
authors: Japhy
tags:
categories:
series:
date: '2020-10-16'
lastmod: '2020-10-16'
draft: false
---



<!--more-->

## 一.分享背景

> &nbsp;&nbsp;&nbsp;go-zero 是一个集成了各种工具实践型的web和rpc框架。通过弹性设计保障了大并发服务端的稳定性，go-zero 包含极简的 API 定义和生成工具 goctl，可以根据定义的 api 文件一键生成 Go, iOS, Android, Kotlin, Dart, TypeScript, JavaScript 代码，并可直接运行。

> &nbsp;&nbsp;&nbsp;分享初心也是为了 Keep前端 也能有一套cli 工具（广义的），小到 一键生成 SFCS组件，API请求，大到微服务应用，脚本等等一切兼并的东西吧。

## 二.分享内容

*本次分享将以demo的形式展现：1. 一个五脏俱全的短链服务； 2. 一个HelloWorld 如何变成kube*

### A.一个五脏俱全的短链服务

#### 1. goctl api

我们先用 goctl 生成一个简单的 api 服务吧

```shell
  goctl api -o shorturl.api
```

### B.一个HelloWorld如何变成kube