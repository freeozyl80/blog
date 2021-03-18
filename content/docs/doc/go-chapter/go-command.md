---
title: go基础命令知识
description: go基础命令知识
toc: true
authors:
tags:
categories:
series:
date: '2020-10-16'
lastmod: '2020-10-16'
draft: false
---

go 基础命令知识

<!--more-->

### 一些常用命令
* go mod 相关：
1）go mod init // 初始化模块名
2) go mod edit // 编辑go.mod文件
3) go mod tidy // 删除错误或者不使用的mod
4) go get -d -v ./ //安装所有依赖
* go test 相关
> go test 命令，会自动读取源码目录下面名为 *_test.go 的文件，生成并运行测试用的可执行文件
1) 参数:
    * -bench regexp 执行相应的 benchmarks，例如 -bench=.；
    * -cover 开启测试覆盖率
    * -run regexp 只运行 regexp 匹配的函数，例如 -run=Array 那么就执行包含有 Array 开头的函数；
    * -v 显示测试的详细命令。
2