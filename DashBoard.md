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

docker run -d \
--restart=always \
--name rmqnamesrv \
-p 9876:9876 \
-v /docker/rocketmq/data/namesrv/logs:/root/logs \
-v /docker/rocketmq/data/namesrv/store:/root/store \
-e "MAX_POSSIBLE_HEAP=100000000" \
rocketmqinc/rocketmq:4.2.0 \
sh mqnamesrv 

docker run -d  \
--restart=always \
--name rmqbroker \
--link rmqnamesrv:namesrv \
-p 10911:10911 \
-p 10909:10909 \
-v  /docker/rocketmq/data/broker/logs:/root/logs \
-v  /docker/rocketmq/data/broker/store:/root/store \
-v /docker/rocketmq/conf/broker.conf:/opt/rocketmq-4.2.0/conf/broker.conf \
-e "NAMESRV_ADDR=namesrv:9876" \
-e "MAX_POSSIBLE_HEAP=200000000" \
rocketmqinc/rocketmq:4.2.0 \
sh mqbroker -c /opt/rocketmq-4.2.0/conf/broker.conf 

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
