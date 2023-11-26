broker 提供消息存储

# 存储文件布局

## commitlog
消息存储文件，所有主题的消息都存储在 commitlog 文件中，默认每个 commitlog 文件大小为 1GB，文件命名按照存储在该消息文件中的第一条消息对应的全局偏移量来命名文件，方便根据消息的物理偏移量进行二分查找，这是 RocketMQ 实现快速检索的基石。


## consumequeue
消息消费队列文件，是 commitlog 文件基于主题的索引文件，主要用于消费者根据主题消费消息。按照 /topic/queue 的形式组织，第一级子目录为主题，第二级子目录为队列，每个队列拥有一个文件夹。

ConsumeQueue 文件长度为 300000 \* 20，如果将文件全部加载到内存，可以将 ConsumeQueue 看作一个数组，每个元素占用 20 个字节，可以实现高效访问。存储格式如下：
- 8个字节的 commitlog 偏移量，即消息的物理偏移量。
- 4个字节的消息长度。
- 8个字节的tag
comsumequeue 本质上就是 commitlog 文件创建的索引文件，其下标就是在 RocketMQ 中的逻辑偏移量，用 queueOffset 表示，消费端消息进度中存储的消费位置就是 queueOffset。


## index
Hash 索引文件，RocketMQ 支持按照消息属性查找消息，实现机制是为需要查找的属性创建 Hash 索引。

## config
broker 运行时配置文件，包含了：
- topics.json：当前 broker 中主题的路由信息。
- subscriptionGroup.json：消费组的订阅关系。
- consumerOffset.json：消费组在集群模式下的消息消费进度文件。
- consumerFilter.json：基于 SQL 92 表达式过滤的过滤规则存储文件
- delayOffset.json：延迟队列处理进度文件。

## lock
RocketMQ 存储目录文件锁，避免启动的多个进程占用同一个存储目录。

## checkPoint
检测点文件，用于记录 commitlog、ConsumerQueue、Index 文件的刷盘点。

## abort
用于判断 RocketMQ 是正常退出还是异常退出，该文件在启动时创建，在正常退出之前删除。

# 存储机制

## commitlog 顺序写

类似于 MySQL redolog，RocketMQ 为了高性能也采用了顺序写，所有主题的消息都按照到达顺序追加到 commitlog 文件中，极大的降低了磁盘写入延迟。

## 内存映射与页缓存

commitlog 被设计为定长大小的文件，主要是为了方便进行内存映射，将磁盘文件映射到内存，以访问内存的方式访问磁盘，极大的提高了文件的操作性能（[[零拷贝技术]]）

内存映射将磁盘数据映射到内存，也就是向内存映射中写入数据，这些数据并不会立即同步到磁盘，需要定时刷盘或者由操作系统来决定。因此如果 RocketMQ 异常退出，操作系统会保证页缓存不丢失，但是如果是机器断电这种情况就有可能会丢失。

## 内核读写分离
RocketMQ 基于内存映射机制，所以消息写入和读取都严重依赖于 Page Cache，如果 Page Cache 发生抖动，那么就会影响性能。因此，引入了堆外内存，通过 transientStorePoolEnable 开关设置。

开启后，消息写入将只写入堆外内存，然后开启定时线程将数据按批次提交到 FileChannel 中，并定时刷写到磁盘，从而大大减少 Page Cache 的写操作次数。消息拉取会从 Page Cache 中读取数据，如果未命中则通过缺页中断，将数据从磁盘加载到 Page Cache 中，实现了读写分离。

## 刷盘机制

### 同步刷盘
同步刷盘，broker 端在收到消息发送者的消息之后，将内容写入内存并持久化到磁盘之后才向客户端返回消息发送成功。
但是也并不是一次只刷一条消息，而是以组提交的形式刷一批消息。

### 异步刷盘
同步刷盘虽然可靠，但是牺牲了较多的性能。
异步刷盘会将消息先存入 PageCache，一旦写入内存，就立即返回消息发送结果。那么如果操作系统崩溃就有可能丢失消息。如果业务系统能容忍一定概率的消息丢失，或者可以通过较低成本的方式找回消息的话，建议使用异步刷盘。

## 复制机制



## 文件恢复

ConsumeQueue 文件和 Index 文件都可以看作是 commitlog 的索引文件，并且 RocketMQ 使用了异步构建的策略，这就有可能导致 commitlog 与 ConsumeQueue 和 Index 文件之间的数据不一致。

此时就可以通过文件恢复机制来实现最终一致性。
RocketMQ 在刷盘时