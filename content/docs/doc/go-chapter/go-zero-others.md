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

1> 我们先用 goctl 生成一个简单的 api 服务吧

```shell
  goctl api -o shorturl.api #生成模板
```
*可以直观的看到两个请求出来*
```shell
  goctl api go -api shorturl.api -dir . #生成代码
```
*可以看到整体代码框架*
```shell
  go run shorturl.go -f etc/shorturl-api.yaml  #run起来
  curl -i "http://localhost:8888/shorten?url=http://www.keep.com"
```

2> 我们需要一个transform rpc服务
```shell 生成proto文件
goctl rpc template -o transform.proto #生成模板
```
*两个rpc的请求已经出来了*
```shell
goctl rpc proto -src transform.proto -dir . #生成code
```
``` shell
#修改模板
syntax = "proto3";

package transform;

message expandReq {
    string shorten = 1;
}

message expandResp {
    string url = 1;
}

message shortenReq {
    string url = 1;
}

message shortenResp {
    string shorten = 1;
}

service transformer {
    rpc expand(expandReq) returns(expandResp);
    rpc shorten(shortenReq) returns(shortenResp);
}
```
*code化*
```shell
ectd #打开etcd
go run transform.go -f etc/transform.yaml #run起来
etcdctl get transform.rpc --prefix #检测是否运行
```
3> 我们将 transform 的rpc 交由 api 去调用
* 修改shorturl-api.yaml,增加rpc调用
```yaml
Transform:
  Etcd:
    Hosts:
      - localhost:2379
    Key: transform.rpc
```
*通过etcd自动发现可用的transform服务*
* 修改 internal/config/config.go 如下，增加 transform 服务依赖
```go
type Config struct {
  rest.RestConf
  Transform zrpc.RpcClientConf     // 手动代码
}
```
* 修改 internal/svc/servicecontext.go，如下：

```go
type ServiceContext struct {
  Config    config.Config
  Transformer transformer.Transformer                                          // 手动代码
}

func NewServiceContext(c config.Config) *ServiceContext {
  return &ServiceContext{
    Config:    c,
    Transformer: transformer.NewTransformer(zrpc.MustNewClient(c.Transform)),  // 手动代码
  }
}
```
*通过 ServiceContext 在不同业务逻辑之间传递依赖*

* 修改 internal/logic/expandlogic.go 里的 Expand 方法，如下：
```go
func (l *ExpandLogic) Expand(req types.ExpandReq) (types.ExpandResp, error) {
  // 手动代码开始
  resp, err := l.svcCtx.Transformer.Expand(l.ctx, &transformer.ExpandReq{
  	Shorten: req.Shorten,
  })
  if err != nil {
  	return types.ExpandResp{}, err
  }

  return types.ExpandResp{
  	Url: resp.Url,
  }, nil
  // 手动代码结束
}
```
* 修改 internal/logic/shortenlogic.go，如下：

``` go
func (l *ShortenLogic) Shorten(req types.ShortenReq) (types.ShortenResp, error) {
  // 手动代码开始
  resp, err := l.svcCtx.Transformer.Shorten(l.ctx, &transformer.ShortenReq{
  	Url: req.Url,
  })
  if err != nil {
  	return types.ShortenResp{}, err
  }

  return types.ShortenResp{
  	Shorten: resp.Shorten,
  }, nil
  // 手动代码结束
}

```
4> 我们现在开始 依赖DB 和 Redis吧

* 定义表结构
 rpc/transform/model -> shorturl.sql
```sql
CREATE TABLE `shorturl`
(
  `shorten` varchar(255) NOT NULL COMMENT 'shorten key',
  `url` varchar(255) NOT NULL COMMENT 'original url',
  PRIMARY KEY(`shorten`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8mb4;
```
 前置条件 
```shell
create database gozero;
source shorturl.sql;
```

* goctl 自动生成 curd+cache代码

```shell
goctl model mysql ddl -c -src shorturl.sql -dir . #根据sql模板生成代码
```

transform配置增加
```
DataSource: root:zhangyunlu@tcp(localhost:3306)/gozero
Table: shorturl
Cache:
  - Host: localhost:6379
```
修改 rpc/transform/internal/config/config.go，如下：
```go
type Config struct {
  zrpc.RpcServerConf
  DataSource string             // 手动代码
  Table      string             // 手动代码
  Cache      cache.CacheConf    // 手动代码
}
```

增加mysql 和 redis cache配置

servicecontext.go
```go
type ServiceContext struct {
  c     config.Config
  Model model.ShorturlModel   // 手动代码
}

func NewServiceContext(c config.Config) *ServiceContext {
  return &ServiceContext{
    c:             c,
    Model: model.NewShorturlModel(sqlx.NewMysql(c.DataSource), c.Cache), // 手动代码
  }
}
```

expandlogic.go
```go 
func (l *ExpandLogic) Expand(in *transform.ExpandReq) (*transform.ExpandResp, error) {
  // 手动代码开始
  res, err := l.svcCtx.Model.FindOne(in.Shorten)
  if err != nil {
    return nil, err
  }

  return &transform.ExpandResp{
    Url: res.Url,
  }, nil
  // 手动代码结束
}
```

shortenlogic.go
```go
func (l *ShortenLogic) Shorten(in *transform.ShortenReq) (*transform.ShortenResp, error) {
  // 手动代码开始，生成短链接
  key := hash.Md5Hex([]byte(in.Url))[:6]
  _, err := l.svcCtx.Model.Insert(model.Shorturl{
    Shorten: key,
    Url:     in.Url,
  })
  if err != nil {
    return nil, err
  }

  return &transform.ShortenResp{
    Shorten: key,
  }, nil
  // 手动代码结束
}
```

重新启动看看

启动transform
启动api

``` shell
curl -i "http://localhost:8888/shorten?url=http://www.keep.com"
curl -i "http://localhost:8888/expand?shorten=dcb613"
```



### B.一个HelloWorld如何变成kube

* 我们简单先起一个服务
```shell
goctl api new hello
go run hello.go -f etc/hello-api.yaml
curl -i "http://localhost:8888/from/me"
```

* 我们把它打包成一个镜像
```shell
goctl docker -go hello.go
docker build -t hello:v2 -f ./hello/Dockerfile .
docker run --rm -it -p 8888:8888 hello:v2
```

* 我们把它放在 k8s 里面跑起来

这里使用 minikube作为 k8s 集群
```shell
kubectl config view
```

因为 minikube 集群里面 不能 拉得主机的 源，所以这里配置了本地私有源，通过网络共享过去

``` shell
docker pull registry
docker run -d -p 5000:5000 -v $(pwd):/var/lib/registry --restart always --name registry registry:2
docker tag hello:v2 10.2.17.5:5000/hello-v2
docker push 10.2.17.5:5000/hello-v2
```

补充配置
```shell
# docker engine
{
  "experimental": false,
  "insecure-registries": [
    "10.2.17.5:5000"
  ],
  "debug": true
}
```

启动minikube
```shell
minikube start --cpus=2 --memory=4096 --disk-size=10g \
  --driver=hyperkit \
  --registry-mirror=https://registry.docker-cn.com \
  --insecure-registry=10.2.17.5:5000 \
  --service-cluster-ip-range='10.10.0.0/24'
```

部署deployment
```shell
kubectl create namespace keep
goctl kube deploy -name hello -namespace keep -image 10.2.17.5:5000/hello-v2 -o hello.yaml -port 8888
kubectl apply -f hello.yaml
kubectl get pods -n keep
```

deployment 生成service
```shell
kubectl expose deployment -n keep hello --port=8888 --target-port=8888 --type=NodePort
minikube service list
curl -i 'http://192.168.64.5:32051/from/you'
```

C. 用 goctl 生成ts代码

```shell
goctl api go -api hello.api -dir .
```