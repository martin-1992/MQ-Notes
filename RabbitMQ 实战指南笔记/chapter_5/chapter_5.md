
### RabbitMQ 管理
　　每个 RabbitMQ 服务器能创建虚拟的消息服务器，即虚拟主机（vhost）。每一个 vhost 本质上是一个独立的小型 RabbitMQ 服务器，拥有自己独立的队列、交换器及绑定关系等。可为不同的业务功能、场景划分不同的 vhost 来使用，vhost 保证绝对隔离。<br />
　　vhost 是 AMQP 概念的基础，客户端在连接时必须制定一个 vhost。
