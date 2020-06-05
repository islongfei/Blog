# canal
canal 主要用途是基于 MySQL 数据库增量日志解析，提供增量数据订阅和消费。
### mysql 主从复制原理
<img src="https://github.com/islongfei/Blog/blob/master/images/mysql%E4%B8%BB%E4%BB%8E%E5%A4%8D%E5%88%B6.jpg"  />  

* MySQL master 将数据变更写入二进制日志( binary log, 其中记录叫做二进制日志事件binary log events，可以通过 show binlog events 进行查看)
* MySQL slave 将 master 的 binary log events 拷贝到它的中继日志(relay log)
* MySQL slave 重放 relay log 中事件，将数据变更反映它自己的数据
### canal 工作原理
* canal 模拟 MySQL slave 的交互协议，**伪装自己为 MySQL slave** ，向 MySQL master 发送dump 协议
* MySQL master 收到 dump 请求，开始推送 binary log 给 slave (即 canal )
* canal 解析 binary log 对象(原始为 byte 流)

支持将canal server接收到的binlog数据直接投递到 RocketMQ Kafka 中。
