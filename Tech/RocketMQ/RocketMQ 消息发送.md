# 消息发送 API

![[Pasted image 20231113235952.png]]

DefaultMQProducer 实现了 MQProducer 接口，MQProducer 继承了 MQAdmin 接口。
MQProducer 是消息发送接口，MQAdmin 是集群管理基础接口，ClientConfig 类是客户端配置类。


## 发送接口分类

按照发送方式
- 同步发送
- 异步发送：异步回调发送结果
- 一次发送：无结果返回

按一次发送消息数量分类
- 单条消息发送
- 批量消息发送

按照是否指定队列发送
- 随机选择发送
- 指定特定消息队列
- 自定义消息队列选择器

## 集群管理接口

MQAdmin 提供了对集群的基础管理能力，也提供了主题创建和消息查询等功能。

## 客户端配置说明

## DefaultMQProducer
默认发送类，使用最为广泛。

