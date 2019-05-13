# RocketMQ Broker

## 概述

1. Broker即是物理上的概念，也是逻辑上的概念。多个物理Broker通过IP:PORT区分，多个逻辑Broker通过BrokerName区分
1. 多个逻辑Broker组成Cluster。
1. Broker与Topic是多对多的关系。
1. Broker自身包含一个使用10911端口的NettyServer、一个10909的NettyServer，以及一个NettyClient。
1. HA通过10912端口提供服务，用于Broker内部各个部分的数据传输。
1. Broker是最重要的部分，包括持久化消息、Broker集群一致性(HA)、保存历史消息、保存Consumer消费偏移量、索引创建等。
1. Producer发送来的消息最终会通过CommitLog序列化到硬盘，执行序列化逻辑的类为AppendMessageCallback接口的实现类。
1. Broker序列化消息是顺序写，序列化文件保存在userHome/store/commitlog目录下，文件名为总偏移量。
1. 默认为异步刷盘、提交日志单个文件1个G、单个consumer队列文件为不到6M

## broker initialize 方法

![avatar](/学习知识\图片\rocketmq\源码\BrokerStartup.createBrokerController.png)

> 配置文件读取

通过命令读取配置文件或则配置参数的信息解析成BrokerConfig,NettyServerConfig,NettyClientConfig,MessageStoreConfig

>org.apache.rocketmq.broker.BrokerController#initialize 初始化

资源初始化

* topicConfigManager.load();
* consumerOffsetManager.load();
* subscriptionGroupManager.load();
* consumerFilterManager.load();
* messageStore.load();
* remotingServer 加载

registerProcessor:注册Netty的处理器

* SendMessageProcessor
* PullMessageProcessor
* QueryMessageProcessor
* ClientManageProcessor
* ConsumerManageProcessor
* EndTransactionProcessor
* AdminBrokerProcessor

定时处理器相关的
打印一天消息处理情况
定时刷新consumerOffsetManager,consumerFilterManager文件序列化
定时检查org.apache.rocketmq.common.stats.MomentStatsItem#value大于速度临界值来禁用慢的consumer来保护Broker
定时打印核心线程的队列长度及队列头的堆积时间
定时打印获取已存储在提交日志中但尚未分配给使用队列的字节数
从服务同步
this.syncTopicConfig();
this.syncConsumerOffset();
this.syncDelayOffset();
this.syncSubscriptionGroupConfig();
主服务打印从服务消息存储落后的长度

开启事物服务org.apache.rocketmq.broker.transaction.TransactionalMessageService
两阶段提交服务

## broker shutdown 方法

![avatar](/学习知识\图片\rocketmq\源码\BrokerShutdown.png)

主要对brokerStatsManager,clientHousekeepingService,pullRequestHoldService,remotingServer,fastRemotingServer,fileWatchService,messageStore,sendMessageExecutor,pullMessageExecutor,adminBrokerExecutor,brokerOuterAPI,consumerOffsetManager,filterServerManager,brokerFastFailure,consumerFilterManagerm,clientManageExecutor,queryMessageExecutor,consumerManageExecutor,fileWatchService 线程池,网络协议,内部服务资源的关闭。