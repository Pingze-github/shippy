## shippy

[Golang 微服务系列教程](https://wuyin.io/tags/%E5%BE%AE%E6%9C%8D%E5%8A%A1/) 源码，每一节对应一个分支。

欢迎提 issue 加以改进，十分感谢 :)


--------------

## 环境配置

### 安装golang
```
wget https://dl.google.com/go/go1.11.5.linux-amd64.tar.gz
tar -xzvf go1.11.5.linux-amd64.tar.gz -C /usr/local
```

### 配置环境变量

假如设定GOPATH是~/go。

```
export GOPATH=~/go
export GOROOT=/usr/local/go
export PATH=$PATH:$GOROOT/bin:$GOPATH/bin
```

### 预装一些被墙的常用包

```
git clone https://github.com/grpc/grpc-go.git $GOPATH/src/google.golang.org/grpc
git clone https://github.com/golang/net.git $GOPATH/src/golang.org/x/net
git clone https://github.com/golang/text.git $GOPATH/src/golang.org/x/text
git clone https://github.com/golang/crypto.git $GOPATH/src/golang.org/x/crypto
go get -u github.com/golang/protobuf/{proto,protoc-gen-go}
git clone https://github.com/google/go-genproto.git $GOPATH/src/google.golang.org/genproto
```
