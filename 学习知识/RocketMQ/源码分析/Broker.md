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

## broker启动流程

broker启动过程中通过 指令  -c 读取配置文件,初始化BrokerConfig,NettyServerConfig,NettyClientConfig,MessageStoreConfig的配置
创建BrokerController,在创建BrokerController的过程中出事化ConsumerManager,ConsumerOffsetManager


