---
layout: post
title:  "一个RPC框架—Dubbo"
categories: Dubbo 
tags: Dubbo 分布式 RPC 
author: sukbear
---

* content
{:toc}
## Dubbo

Dubbo是一款高性能、轻量级的开源Java RPC 框架,它提供了三大核心能力：面向接口的远程方法调用，智能容错和负载均衡，以及服务自动注册和发现。

#### Maven dependency
```bash
<properties>
    <dubbo.version>2.7.0</dubbo.version>
</properties>
    
<dependencyManagement>
    <dependencies>
        <dependency>
            <groupId>org.apache.dubbo</groupId>
            <artifactId>dubbo-dependencies-bom</artifactId>
            <version>${dubbo.version}</version>
            <type>pom</type>
            <scope>import</scope>
        </dependency>
    </dependencies>
</dependencyManagement>

<dependencies>
    <dependency>
        <groupId>org.apache.dubbo</groupId>
        <artifactId>dubbo</artifactId>
        <version>${dubbo.version}</version>
    </dependency>
    <dependency>
        <groupId>io.netty</groupId>
        <artifactId>netty-all</artifactId>
    </dependency>
    <dependency>
        <groupId>org.apache.curator</groupId>
        <artifactId>curator-framework</artifactId>
    </dependency>
    <dependency>
        <groupId>org.apache.curator</groupId>
        <artifactId>curator-recipes</artifactId>
    </dependency>
    <dependency>
        <groupId>org.apache.zookeeper</groupId>
        <artifactId>zookeeper</artifactId>
    </dependency>
</dependencies>
```

### RPC (Remote Procedure Call)
- 远程过程调用，它是一种通过网络从远程计算机程序上请求服务，而不需要了解底层网络技术的协议。
   - 服务消费方（client）调用以本地调用方式调用服务；
   - client stub接收到调用后负责将方法、参数等组装成能够进行网络传输的消息体；
   - client stub找到服务地址，并将消息发送到服务端；
   - server stub收到消息后进行解码；
   - server stub根据解码结果调用本地的服务；
   - 本地服务执行并将结果返回给server stub；
   - server stub将返回结果打包成消息并发送至消费方；
   - client stub接收到消息，并进行解码；
   - 服务消费方得到最终结果。
   
   
### Dubbo架构
![git](https://raw.githubusercontent.com/sukbear/sukbear.github.io/master/images/dubbo.jpg)
- Provider： 暴露服务的服务提供方
- Consumer： 调用远程服务的服务消费方
- Registry： 服务注册与发现的注册中心
- Monitor： 统计服务的调用次数和调用时间的监控中心
- Container： 服务运行容器
- 调用关系说明：
    - 服务容器负责启动，加载，运行服务提供者。
    - 服务提供者在启动时，向注册中心注册自己提供的服务。
    - 服务消费者在启动时，向注册中心订阅自己所需的服务。
    - 注册中心返回服务提供者地址列表给消费者，如果有变更，注册中心将基于长连接推送变更数据给消费者。
    - 服务消费者，从提供者地址列表中，基于软负载均衡算法，选一台提供者进行调用，如果调用失败，再选另一台调用。
    - 服务消费者和提供者，在内存中累计调用次数和调用时间，定时每分钟发送一次统计数据到监控中心。
 
    
#### 关系调用
  - 服务容器负责启动，加载，运行服务提供者。
  - 服务提供者在启动时，向注册中心注册自己提供的服务。
  - 服务消费者在启动时，向注册中心订阅自己所需的服务。
  - 注册中心返回服务提供者地址列表给消费者，如果有变更，注册中心将基于长连接推送变更数据给消费者。
  - 服务消费者，从提供者地址列表中，基于软负载均衡算法，选一台提供者进行调用，如果调用失败，再选另一台调用。
  - 服务消费者和提供者，在内存中累计调用次数和调用时间，定时每分钟发送一次统计数据到监控中心。
  
  
### 工作原理
 ![git](https://raw.githubusercontent.com/sukbear/sukbear.github.io/master/images/dubbo1.jpg)
    图中从下至上分为十层，各层均为单向依赖，右边的黑色箭头代表层之间的依赖关系，每一层都可以剥离上层被复用，其中，Service 和 Config 层为 API，其它各层均为 SPI。
- 各层说明：
    第一层：service层，接口层，给服务提供者和消费者来实现的
    第二层：config层，配置层，主要是对dubbo进行各种配置的
    第三层：proxy层，服务接口透明代理，生成服务的客户端 Stub 和服务器端 Skeleton
    第四层：registry层，服务注册层，负责服务的注册与发现
    第五层：cluster层，集群层，封装多个服务提供者的路由以及负载均衡，将多个实例组合成一个服务
    第六层：monitor层，监控层，对rpc接口的调用次数和调用时间进行监控
    第七层：protocol层，远程调用层，封装rpc调用
    第八层：exchange层，信息交换层，封装请求响应模式，同步转异步
    第九层：transport层，网络传输层，抽象mina和netty为统一接口
    第十层：serialize层，数据序列化层。网络传输需要。
   
    
### 负载均衡策略
    负载均衡改善了跨多个计算资源（例如计算机，计算机集群，网络链接，中央处理单元或磁盘驱动的的工作负载分布。负载平衡旨在优化资源使用，最大化吞吐量，最小化响应时间，并避免任何单个资源的过载。使用具有负载平衡而不是单个组件的多个组件可以通过冗余提高可靠性和可用性。负载平衡通常涉及专用软件或硬件
-  Random LoadBalance(默认，基于权重的随机负载均衡机制)
    - 随机，按权重设置随机概率。
    - 在一个截面上碰撞的概率高，但调用量越大分布越均匀，而且按概率使用权重后也比较均匀，有利于动态调整提供者权重。
- ConsistentHash LoadBalance
    - 一致性 Hash，相同参数的请求总是发到同一提供者。(如果你需要的不是随机负载均衡，是要一类请求都到一个节点，那就走这个一致性hash策略。)
    - 当某一台提供者挂时，原本发往该提供者的请求，基于虚拟节点，平摊到其它提供者，不会引起剧烈变动。
    
    
### zookeeper宕机与dubbo直连的情况
 在实际生产中，假如zookeeper注册中心宕掉，一段时间内服务消费方还是能够调用提供方的服务的，实际上它使用的本地缓存进行通讯，这只是dubbo健壮性的一种提现。
- dubbo的健壮性表现：
    - 监控中心宕掉不影响使用，只是丢失部分采样数据
    - 数据库宕掉后，注册中心仍能通过缓存提供服务列表查询，但不能注册新服务
    - 注册中心对等集群，任意一台宕掉后，将自动切换到另一台
    - 注册中心全部宕掉后，服务提供者和服务消费者仍能通过本地缓存通讯
    - 服务提供者无状态，任意一台宕掉后，不影响使用
    - 服务提供者全部宕掉后，服务消费者应用将无法使用，并无限次重连等待服务提供者恢复
    - 注册中心负责服务地址的注册与查找，相当于目录服务，服务提供者和消费者只在启动时与注册中心交互，注册中心不转发请求，压力较小。所以，我们可以完全可以绕过注册中心——采用 dubbo 直连 ，即在服务消费方配置服务提供方的位置信息。
    
    

