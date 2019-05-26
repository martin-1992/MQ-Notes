
## RabbitMQ 配置
　　三种配置方式：环境变量、配置文件、运行时参数和策略设置。
  
### 优化网络配置
　　网络是客户端和 RabbitMQ 之间通信的媒介，RabbitMQ 支持的所有协议都是基于 TCP 层面的，所有的 RabbitMQ 设置都可在 rabbitmq.config 配置文件中配置实现。<Br />
　　优化网络配置的一个重要目标就是提高吞吐量：
  
  
- 禁用 Nagle 算法，主要减少延迟。因为 Nagle 算法是会合并消息后发送，禁用后小消息收到也会立即发送，[Netty 也是禁用了 Nagle 算法](https://github.com/martin-1992/Netty_analysis/blob/master/%E6%96%B0%E8%BF%9E%E6%8E%A5%E7%9A%84%E6%8E%A5%E5%85%A5/NioSocketChannel%20%E7%9A%84%E5%88%9B%E5%BB%BA.md)；
- 增大 TCP 缓冲区的大小，提高吞吐量。每个 TCP 连接都分配了缓冲区，一般来说，缓冲区越大，吞吐量也越高，但是每个连接上耗费的内存也就越多，从而使总体服务的内存增大。而减少 TCP 缓冲区大小，提高文件句柄数，则可提高并发量。
