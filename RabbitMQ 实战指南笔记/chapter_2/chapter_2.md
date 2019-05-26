
## RabbitMQ 入门
　　RabbitMQ 整体上是一个生产者与消费者模型，主要负责接收、存储和转发消息。
  
### 生产者和消费者

- Producer，生产者创建消息，然后投递到 RabbitMQ 中。消息包含 2 个部分：消息体和标签。消息体是带有业务逻辑结构的数据，比如一个 JSON 字符串。而消息的标签用来描述这条消息（类似于娱乐标签、体育消息标签等），比如一个交换器的名称和一个路由键（娱乐标签的消息发给名称为娱乐的交换器，不同类型的消费者则根据交换器的名称标签来获取处理不同类型的消息）。RabbitMQ 根据标签把消息发送给感兴趣的消费者；
- Consumer，消费者接收消息。消费者连接到 RabbitMQ 服务器，并订阅到队列上；
- Broker，消费中间件的服务节点，一个 RabbitMQ Broker 可看作一个 RabbitMQ 服务节点。


　　下图为生产者将消息存入 RabbitMQ Broker，消费者从 Broker 中消费数据的流程：
  
![image.png](attachment:image.png)

### 队列
　　队列，是 RabbitMQ 的内部对象，用于存储消息。**多个消费者可订阅同一个队列，这时队列中的消费会被平均分摊给多个消费者进行处理，而不是每个消息者都收到所有的消息。**

### 交换器
　　生产者将消息发送到 Exchange（交换器），由交换器将消息路由到一个或多个队列中，路由不到，则返回给生产者或丢弃。
  
![image.png](attachment:image.png)

### 路由键
　　RoutingKey，路由键。生产者将消息发给交换器时，会指定一个路由键，用来指定这个消息的路由规则，这个路由键配合交换器类型和绑定键一起使用才能生效。<br />
　　在交换器类型和绑定键固定的情况下，生产者可在发送消息给交换器时，通过指定路由键来决定消息发到谁。
  
### 绑定键
　　RabbitMQ 通过绑定将交换器与队列关联起来，绑定键对应队列，绑定键可相同。如下图：
  
![image.png](attachment:image.png)

### 交换器、路由键、绑定键三者的关键
　　交换器相当于投递包裹的邮箱，路由键相当于填写在包裹上的地址，绑定键相当于包裹的目的地。当填写在包裹上的地址和实际想要投递的地址相匹配时，包裹才会被正确投递到目的地，即队列中。<br />
　　如果地址出错，不能正确投递，则队列会退回或丢弃。
  
### 交换器类型

- fanout，把所有发送到该交换器的消息路由到所有与该交换器绑定的队列中；
- direct，把消息路由到那些路由键和绑定键完全匹配的队列中。如下图，设置路由键为“debug”，消息只会路由到队列 2，设置为“warning”，则路由到队列 1 和队列 2；

![image.png](attachment:image.png)

- topic，按规则匹配。如下图，有三种绑定键为：\*.rabbitmq.\*、\*.*.client、com.#。路由键为“com.hidden.client”的消息路由到队列 2，路由键为“java.rabbitmq.demo”的消息路由到队列1，路由键为“java.util.concurrent”的消息没有匹配的，会被丢弃或返回给生产者；

![image.png](attachment:image.png)

- headers，根据发送的消息内容中的 headers 属性来匹配，性能很差，很少用。

### 生产者发送消息的流程

- 生产者连接到 RabbitMQ Broker，建立一个连接，开启一个信道（Channel），多个信道复用一个连接（多路复用）；
- 生产者声明一个交换器，并设置相关属性，比如交换器类型、是否持久化等；
- 生产者声明一个队列并设置相关属性，比如是否排他、是否持久化、是否自动删除等；
- 生产者通过路由键将交换器和队列绑定起来；
- 生产者发送消息至 RabbitMQ Broker，其中包含路由键、交换器等信息；
- 相应的交换器根据接收到的路由键查找相匹配的（绑定键）队列；
- 找到，则将从生产者发送过来的消息存入相应的队列中；
- 没找到，则根据生产者配置的属性选择丢弃还是回退给生产者；
- 关闭信道，关闭连接。

### 消费者接收消息的过程

- 消费者连接到 RabbitMQ Broker，建立一个连接，开启一个信道；
- 消费者向 RabbitMQ Broker 请求消费相应队列中的消息，可能会设置相应的回调函数，以及做一些准备工作；
- 等待 RabbitMQ Broker 回应并投递相应队列中的消息，消费者接收消息；
- 消费者确认（ACK）接收到的消息；
- RabbitMQ 从队列中删除相应已经被确认的消息；
- 关闭信道，关闭连接。

### 连接和信道
　　生产者和消费者都需要和 RabbitMQ Broker 建立 TCP 连接，当连接建立后，客户端再创建一个 AMQP 信道，每个信道都会指派一个唯一的 ID。信道是建立在连接之上的虚拟连接，一个 TCP 连接有多个信道（连接），RabbitMQ 处理的每条 AMQP 指令是通过信道完成的。

![image.png](attachment:image.png)

　　通过在一个 TCP 连接上建立多个信道，可复用 TCP 连接，减少性能开销，因为建立和销毁 TCP 连接是昂贵的开销。<br />

### AMQP 协议介绍
　　RabbitMQ 是遵从 AMQP 协议的，即 RabbitMQ 是 AMQP 协议的 Erlang 的实现。AMQP 的模型架构和 RabbitMQ 的模型是一样的，流程也是一样的。<br />
　　AMQP 协议是一个通信协议，可将 AMQP 协议看做一系列结构化的命令集合，这里的命令表示一种操作，类似于 HTTP 中的方法（GET、POST 等），包括三层：
  
- Module Layer，位于协议最高层。主要定义一些客户端调用的命令，比如客户端使用 Queue.Declare 命令声明一个队列；
- Session Layer，位于中间层。负责将客户端的命令发送给服务器，再将服务器的应答返回给客户端，主要为客户端与服务器之间的通信提供可靠性同步机制和错误处理；
- Transport Layer，位于最底层。传输二进制数据流，提供帧的处理、信道复用、错误检测和数据表示等。

### AMQP 生产者流转过程
　　AMQP 常用的命令与 RabbitMQ 客户端中方法的映射关系，**调用的函数方法会被转换为协议的命令进行发送：**
  
```java
// 创建连接
Connection connection = factory.newConnection();
// 创建信道
Channel channel = connection.createChannel();
String message = "Hello World! ";
channel.basicPublish(EXCHANGE_NAME, ROUTING_KEY , MessageProperties.PERSISTENT_TEXT_PLAIN, message.getBytes());
// 关闭资源
channel.close() ;
connection.close(); 
```
    
- 客户端与 Broker 建立连接时，调用 factory.newConnection 方法，这个方法进一步封装成 Protocol Header 0-9-1 的报文头发送给 Broker，通知 Broker 本次交互采用的是 AMQP 0-9-1 协议；
- Broker 返回 Connection.start 来建立连接，这过程涉及 Connection.Start / .Start-Ok、Connection.Tune / .Tune-Ok、Connection.Open / .Open-Ok 这 6 个命令的交互；
- 当客户端调用 connection.createChannel 方法准备开启信道的时候，其包装 Channel.Open 命令发送给 Broker，等待 Channel.Open-Ok 命令；
- 当客户端发送消息时，会调用 channel.basicPublish 方法，对应的 AQMP 命令为 Basic.Publish；
- 客户端关闭资源时，涉及 Channel.Close / .Close-Ok 与 Connection.Close / .Close-Ok 的命令交互。

　　下图为流转过程：
  
![image.png](attachment:image.png)
