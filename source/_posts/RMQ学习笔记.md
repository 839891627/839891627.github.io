---
title: RMQ学习笔记
date: 2017-07-03 11:28:33
categories: 编程
tags: 
    - RMQ
    - 消息队列
---

#### 路由绑定以及消息路由
![rmq](./rmq.jpg)

<!-- more -->

##### 发布者：
```flow
st=>start: 开始
e=>end: 结束
op1=>operation: 连接到RabbitMQ
op2=>operation: 获取信道 channel
op3=>operation: 声明交换器 exchange
op4=>operation: 创建消息
op5=>operation: 发布消息
op6=>operation: 关闭信道
op7=>operation: 关闭连接

st->op1->op2->op3->op4->op5->op6->op7->e
```

##### 消费者：
```flow
st=>start: 开始
e=>end: 结束
op1=>operation: 连接到RabbitMQ
op2=>operation: 获取信道 channel
op3=>operation: 声明交换器 exchange
op4=>operation: 声明队列
op5=>operation: 把队列和交换器绑定起来
op6=>operation: 消费消息
op7=>operation: 关闭信道
op8=>operation: 关闭连接

st->op1->op2->op3->op4->op5->op6->op7->e
```
#### 函数
1. basic.consume  
> 持续订阅  
2. basic.get  
> 获取一条（会产生订阅/取消订阅的动作，所以不要用循环`basic.get`来取代`basic.consume`）  
3. queue.declare  
> 声明队列，需要传入队列名称，如果不传，则会自动随机生成一个队列名称。  
如果队列名称已经存在，则不会再重复生成，而是返回 true  
> 参数：  
 - exclusive 若为`true`,则队列变成*私有*
 - auto-delete 若为`true`, 则当最后一个消费者取消订阅时，队列会自动删除。 可以配合`exclusive`来使用  
 - passive 用来检测队列是否存在。若为`true`,则会根据队列是否存在，返回 成功/错误，而不是生成新的队列
4. basic_publish ($msg, $exchange, $routing_key)
> 根据 *routing_key* 绑定消息到 *$exchange*上的queue上
5. queue_bind($queue, $exchange, $routing_key)

#### 消息持久化
> 通过将消息写入磁盘日志来持久化  
但是持久化会带来性能上的问题，因为写入硬盘要比写入内存慢得多,所以不建议所有消息都持久化
1. 队列和交换器的`durable`属性设置为`true`
2. 发送到持久化的交换器
3. 到达持久化的队列

[代码笔记](https://github.com/839891627/rmq-notes)
 
