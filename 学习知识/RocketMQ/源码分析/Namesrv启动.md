# Namesrc 源码分析

## Namesrc的作用

namesrv的定位是作为注册中心，保存broker节点的路由信息，保存一些简单的k/v信息。    namesrv支持集群模式，但是每个namesrv之间相互独立不进行任何通信，它的多点容灾通过producer/consumer在访问namesrv的时候轮询获取信息（当前节点访问失败就转向下一个）。    namesrv作为注册中心，负责接收broker定期的注册信息并维持在内存当中，没错namesrv是没有持久化功能的，所有数据都保存在内存当中，broker的注册过程也是循环遍历所有namesrv进行注册。    namesrv通过提供对外接口给producer和consumer访问broker的路由信息，底层通过netty来实现。    namesrv对broker的存活检测机制：心跳机制即namesrv作为broker的server端定期接收broker的心跳信息，超时无心跳就移除broker；连接异常检测机制即底层通过epoll的消息机制来检测连接的断开。

## Namesrv启动整体流程

rocketMq 通过org.apache.rocketmq.namesrv.NamesrvStartup的mian方法传递参数后解析返回org.apache.rocketmq.namesrv.NamesrvControllerNamesrvController调用
initialize做初始化,如果初始化不成功会调用org.apache.rocketmq.namesrv.NamesrvController#shutdown处理。主要关闭初始化流程中创建的资源,最后对JVM关闭增加钩子 NamesrvControllerNamesrvController.shutdown。如果初始化成功调用NamesrvController.start方法

>initialize 处理流程

- org.apache.rocketmq.namesrv.kvconfig.KVConfigManager#load 初始化
- 构建org.apache.rocketmq.remoting.netty.NettyRemotingServer
- 构建org.apache.rocketmq.namesrv.NamesrvController#remotingExecutor 构建远程线程池
- org.apache.rocketmq.namesrv.NamesrvController#registerProcessor 注册netty默认处理器
- 定时扫描处理不活动的broker org.apache.rocketmq.namesrv.routeinfo.RouteInfoManager#scanNotActiveBroker
- 定时打印org.apache.rocketmq.namesrv.kvconfig.KVConfigManager#configTable的值
  
>shutdown 处理流程

- remotingServer.shutdown
- remotingExecutor.shutdown
- scheduledExecutorService.shutdown

>start流程

- org.apache.rocketmq.remoting.RemotingService#start启动 目前只有**netty**的接受
- org.apache.rocketmq.common.ServiceThread#start