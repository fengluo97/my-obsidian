#todo
# 方法论
One By One，兴趣驱动
**第一轮**：学习使用 + 基本原理 + 部分源码 + 应用场景（官方文档 + 视频教程 + 博客 + 书籍）
最主要整理自己的八股文库。
周一、三、五，学习项目经验。其他时间学习八股。

小步快跑

**第二轮**：阅读源码 + 模拟实现 + 开源项目 + 多写多练。

**第二轮并行**：刷题 + 整理项目

周内，每天三个小时。
周末，每天八个小时。

vi /docker/rocketmq/conf/broker.conf
# 所属集群名称，如果节点较多可以配置多个
brokerClusterName = DefaultCluster
#broker名称，master和slave使用相同的名称，表明他们的主从关系
brokerName = broker-a
#0表示Master，大于0表示不同的slave
brokerId = 0
#表示几点做消息删除动作，默认是凌晨4点
deleteWhen = 04
#在磁盘上保留消息的时长，单位是小时
fileReservedTime = 48
#有三个值：SYNC_MASTER，ASYNC_MASTER，SLAVE；同步和异步表示Master和Slave之间同步数据的机制；
brokerRole = ASYNC_MASTER
#刷盘策略，取值为：ASYNC_FLUSH，SYNC_FLUSH表示同步刷盘和异步刷盘；SYNC_FLUSH消息写入磁盘后才返回成功状态，ASYNC_FLUSH不需要；
flushDiskType = ASYNC_FLUSH
# 设置broker节点所在服务器的ip地址
brokerIP1 = 192.168.154.130
# 磁盘使用达到95%之后,生产者再写入消息会报错 CODE: 14 DESC: service not available now, maybe disk full
diskMaxUsedSpaceRatio=95

docker run -d \
--restart=always \
--name rmqadmin \
-e "JAVA_OPTS=-Drocketmq.namesrv.addr=192.168.154.130:9876 \
-Dcom.rocketmq.sendMessageWithVIPChannel=false" \
-p 9999:8080 \
pangliang/rocketmq-console-ng





二月份到三月上旬：**第一轮**刷完所有基础项
![[Pasted image 20240202234757.png]]
三月上旬到四月中旬：
![[Pasted image 20240202235745.png]]
