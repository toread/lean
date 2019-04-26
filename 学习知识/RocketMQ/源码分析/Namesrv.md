# Namesrc 源码分析

## Namesrc的作用

 Name Server作为RocketMQ的一个组件，其作用就是一个注册中心，用于管理Broker相关的一些信息，生产者和消费者可以从Name Server中获取Broker中相关的Topic信息等，Name Server可以单台部署也可以多台部署，相互之间不存在联系。

>Name Server主要有以下四个功能

* 维护一份本地kv配置
* 维护一份Broker信息(集群名称、Broker名称及相关地址信息)，Broker在启动后会将自身信息注册到Name Server中
* 维护每个Topic相关的信息，Broker心跳时会将自身的topic提交到Name Server，生产者在发送消息时会根据Topic名称获取Broker的列表，消费者在监听Topic时也会根据Topic名称从Name Server中获取相关Broker的信息
* 提供对外接口服务(目前默认是netty)  kv配置信息的crud处理,topic信息的查询,broker的注册及注销

## Namesrv启动整体流程

rocketMq 通过org.apache.rocketmq.namesrv.NamesrvStartup的mian方法传递参数后解析返回org.apache.rocketmq.namesrv.NamesrvControllerNamesrvController调用
initialize做初始化,如果初始化不成功会调用org.apache.rocketmq.namesrv.NamesrvController#shutdown处理。主要关闭初始化流程中创建的资源,最后对JVM关闭增加钩子 NamesrvControllerNamesrvController.shutdown。如果初始化成功调用NamesrvController.start方法

>initialize 处理流程

* org.apache.rocketmq.namesrv.kvconfig.KVConfigManager#load 初始化
* 构建org.apache.rocketmq.remoting.netty.NettyRemotingServer
* 构建org.apache.rocketmq.namesrv.NamesrvController#remotingExecutor 构建远程线程池
* org.apache.rocketmq.namesrv.NamesrvController#registerProcessor 注册netty默认处理器
* 定时扫描处理不活动的broker org.apache.rocketmq.namesrv.routeinfo.RouteInfoManager#scanNotActiveBroker
* 定时打印org.apache.rocketmq.namesrv.kvconfig.KVConfigManager#configTable的值
  
>shutdown 处理流程

- remotingServer.shutdown
- remotingExecutor.shutdown
- scheduledExecutorService.shutdown

>start流程

- org.apache.rocketmq.remoting.RemotingService#start启动 目前只**netty**的接受
- org.apache.rocketmq.common.ServiceThread#start

## 注册中心对外提供的接口操作

> kv配置的操作:PUT_KV_CONFIG,GET_KV_CONFIG,DELETE_KV_CONFIG,GET_KVLIST_BY_NAMESPACE

kv的配置通过类KVConfigManager实现的,URD的操作在操作的时候对局部对象进行加锁操作。操作完毕以后同步根据文件,写入文件的方式为Json

>broker的操作:REGISTER_BROKER,UNREGISTER_BROKER,GET_BROKER_CLUSTER_INFO,WIPE_WRITE_PERM_OF_BROKER,UPDATE_NAMESRV_CONFIG,GET_NAMESRV_CONFIG

>topic的操作:GET_ROUTEINTO_BY_TOPIC,GET_ALL_TOPIC_LIST_FROM_NAMESERVER,DELETE_TOPIC_IN_NAMESRV,GET_TOPICS_BY_CLUSTER,GET_SYSTEM_TOPIC_LIST_FROM_NS,GET_UNIT_TOPIC_LIST,GET_HAS_UNIT_SUB_TOPIC_LIST,GET_HAS_UNIT_SUB_UNUNIT_TOPIC_LIST