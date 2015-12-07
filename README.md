# hydrogen-ssdb
Java 编写的 SSDB 客户端

_（底层架构已经完成，正在补充一些具体的命令）_

## 介绍

hydrogen-ssdb 是一个 Java 编写的 [SSDB](https://github.com/ideawu/ssdb)  客户端，支持多线程并发请求和多服务器的负载均衡（客户端分发请求）。

SSDB 是一个类似 Redis 的 NOSQL 数据库，兼容 Redis 协议，且内容占用小。

hydrogen-ssdb 是一个 SSDB 客户端，具有以下特性：

1. 易于配置，易于使用；
1. 支持 SSDB 主从集群与负载均衡；
1. 当集群中的服务器 down 掉时，能自动识别并跳过该服务器。

更多特性正在添加当中。

#### 【负载均衡的拓扑架构】

`Cluster` 是负载均衡的顶层单位，每个 `Cluster` 包含多个 `Server`，一个 `Server` 可以是主服务器，也可以是从服务器，但 `Cluster` 中必须要有主服务器。

![](https://cloud.githubusercontent.com/assets/900606/11584478/2c30724c-9a9f-11e5-8fa2-3917230a227b.png)

##项目依赖

hydrogen-ssdb 依赖于下面两个框架：

* Apache commons-pool2 （对象池框架）
* slf4j （日志框架）

##使用方法

#### 基本使用方法

```java
SsdbClient client = new SsdbClient(host, port);
client.set("key", "value");
```

#### 配置主从服务器
```java
List<Server> servers = Arrays.asList(
        new Server("192.168.1.180", 8888, null, true),  // 主服务器
        new Server("192.168.1.180", 8889, null, false)  // 从服务器
);

SsdbClient client = new SsdbClient(Sharding.fromServerList(servers));
client.set("name", "hydrogen-ssdb");    // 写入请求一定会发送给主服务器
System.out.println(client.get("name")); // 读取请求会随机发送给任意一台服务器
```

#### 配置负载均衡

```java
Sharding sharding = new Sharding(Arrays.asList(
        new Cluster(new Server("192.168.1.180", 8888), 100),  // 100 和 200 这两个参数指的是权重，
        new Cluster(new Server("192.168.1.180", 8889), 200)   // 权重越大的 Cluster 所保存的 key 越多。
));

SsdbClient ssdbClient = new SsdbClient(sharding);

```