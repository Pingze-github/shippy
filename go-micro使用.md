> go-micro plugins: https://github.com/micro/go-plugins

> 参考文章：https://cloud.tencent.com/developer/article/1351761

## 服务发现

可以通过环境变量来指定微服务的端口：
```
MICRO_SERVER_ADDRESS=:50051
MICRO_REGISTRY=mdns
```

同时指定通过mdns多播做服务发现（生产环境可以用Consul）。

不同的微服务需要启动在不同的端口。

## 消息队列插件：Nats

> github.com/micro/go-plugins/broker/nats

nats是一个简单的消息队列应用，类似Kafka，使用消费/订阅模型。

使用：

**#1** 启动一个单实例：
```
docker run -d -p 4222:4222 nats
```

**#2** 在代码中引用
```
import (
    _ "github.com/micro/go-plugins/broker/nats"
)
```

**#3** 在环境变量中设置
```
MICRO_BROKER=nats
MICRO_BROKER_ADDRESS=0.0.0.0:4222
```

