路由注册 & 剔除机制

NameServer 工作流程
1、Broker 每 30 秒向 NameServer 集群的每一台机器发送一次心跳包，包含自身创建的主题路由信息等
2、消息客户端每 30 秒向 NameServer 更新对应主题的路由信息
3、NameServer 在收到 Broker 发送的心跳包时记录时间戳并将其存在 brokerLiveTable 中
4、NameServer 每 10 秒扫描一次 brokerLiveTable，如果在 120秒内没有收到心跳包，则认为 broker 失效，更新主题的路由信息，将失效的 broker 信息移除
5、在网络通信层如果 NameServer 与 Broker 之间的 TCP 连接断开，NameServer 能立即感知 Broker 节点崩溃而不必等待 120 秒，直接删除相关的路由信息

如此以来， NameServer 保证了其设计上的简单性，但是也导致了两个问题：
1、NameServer 不能实时感知路由信息的变化，即便感知到 broker 失效，也不会主动通知消费客户端
2、NameServer 节点之间无状态，相互不通信，走的是依靠 broker 每30秒主动上报信息的最终一致性

无状态导致的问题
- 网络分区：如果 BrokerA 和 NameServerA 与 BrokerB、BrokerC和NameServerB、NameServerC 之间形成了网络分区，那么就会导致 NameServer 中的 broker 信息不一致。
	- 对于消息发送：

