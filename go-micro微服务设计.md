## ***.proto

Protobuf声明文件。

格式类似：
```
syntax = "proto3";

// 服务包名
package go.micro.***;

// 服务声明
service ShippingService {
    rpc Create (User) returns (Response) {
    }
    rpc Get (User) returns (Response) {
    }
    rpc GetAll (Request) returns (Response) {
    }
}

// 信息实体声明
message User {
    string id = 1;
    string name = 2;
    string password = 5;
}

message Response {
    User user = 1;
    repeated User users = 2;
    repeated Error errors = 3;
}

message Error {
    int32 code = 1;
    string description = 2;
}
```


## handler.go

此文件包含所有业务功能。

### 整体结构

所有方法基于结构体：
```
type handler struct {}
```

结构体内，可以声明 数据库连接 或 gRPC客户端，如：
```
type handler struct {
    session *mgo.Session
    vesselClient vesselPb.VesselServiceClient
}
```

方法类似：
```
func (h *handler)GetRepo()Repository  {
    return &ConsignmentRepository{h.session.Clone()}
}
```
即可通过handler对象调用数据库会话。

### API方法参数

API方法类似：
```
func (h *handler) Foo(ctx context.Context, req *pb.GetRequest, resp *pb.Response) error {}
```

参数为：
```
ctx context.Context
req *pb.GetRequest
resp *pb.Response
```

返回值为：
```
err error
```

req/resp都是在proto中定义的。req用于传入参数，resp用于返回参数。

实际不限制req的类型，可以使用proto中定义的其他结构体作为类型。
但定义的类型，在调用时需要保证一致。

如果不需要传入参数，也必须有req。可以定义空的req。

如果需要创建空context，可以用`context.BackGround()`。

* 所有传入参数都需要事先在proto中定义为结构体。

## respository.go

此文件包含所有的数据操作；相当于MVCS中的service。

需要定义个Respository接口，并创建响应的结构体：
```
type Repository interface {
    Create(*pb.Consignment) error
    GetAll() ([]*pb.Consignment, error)
    Close()
}

type ConsignmentRepository struct {
    session *mgo.Session
}
```


可以看到，Repo应该包含数据会话。

Repo应该至少实现 创建，获取，关闭会话的方法。

> 特别注意的是，gopkg.in/mgo.v2，会在connection时生成一个主会话。
之后具体查询时，应根据实际情况，从主会话中Clone()或Copy()出一个新会话来执行。
这样就实现了并发查询。
所以执行完毕后，需要手动关闭一个连接。

> gorm则使用同一个会话，底层由gorm自身调度。

获取`*mgo.collection`的方法是：
```
const (
    DB_NAME        = "shippy"
    CON_COLLECTION = "consignments"
)
collection := repo.session.DB(DB_NAME).C(CON_COLLECTION)
```


## database.go

数据库连接。返回数据库连接会话。

### MongoDB

库：gopkg.in/mgo.v2

不需要指定database。

```
import "gopkg.in/mgo.v2"

func CreateSession(host string) (*mgo.Session, error) {
	s, err := mgo.Dial(host)
	if err != nil {
		return nil, err
	}
	s.SetMode(mgo.Monotonic, true)
	return s, nil
}
```

### PostgreSQL

库：github.com/jinzhu/gorm

需要指定database。

```
import (
    "github.com/jinzhu/gorm"
    _ "github.com/jinzhu/gorm/dialects/postgres"
    "fmt"
    "os"
)

func CreateConnection() (*gorm.DB, error) {
    host := os.Getenv("DB_HOST")
    port := os.Getenv("DB_PORT")
    user := os.Getenv("DB_USER")
    password := os.Getenv("DB_PASSWORD")
    dbName := os.Getenv("DB_NAME")

    return gorm.Open(
        "postgres",
        fmt.Sprintf(
            "host=%s port=%s user=%s dbname=%s sslmode=disable password=%s",
            host, port, user, dbName, password,
        ),
    )
}
```

### Gorm 预设id

[参考文档](http://doc.gorm.io/callbacks.html)

利用Gorm的callbacks机制，可以将SQL数据库的id自动设置为我们需要的uuid。

```
import (
    "github.com/jinzhu/gorm"
    "github.com/satori/go.uuid"
    "github.com/labstack/gommon/log"
)

func (user *User) BeforeCreate(scope *gorm.Scope) error {
    uuid, err := uuid.NewV4()
    if err != nil {
        log.Fatalf("created uuid error: %v\n", err)
    }
    return scope.SetColumn("Id", uuid.String())
}
```

## main.go

各个服务入口文件。

主文件应该负责：
+ DB-Session 创建
+ repo创建（MongoDB不必要）
+ 各个 gRPC-Session 创建
+ 注入 repo/DB-Session 和 gRPC-Session 到handler
+ 创建微服务，注册handler，运行服务

```
func main() {
    // 连接数据库
    db, err := CreateConnection()
    defer db.Close()

    if err != nil {
        log.Fatalf("connect error: %v\n", err)
    }

    // 创建repo
    repo := &UserRepository{db}

    // 微服务创建
    srv := micro.NewService(
        micro.Name("go.micro.srv.user"),
        micro.Version("latest"),
    )

    srv.Init()

    // 注入会话，注册hanlder
    pb.RegisterUserServiceHandler(srv.Server(), &handler{repo})

    // 启动服务
    if err := srv.Run(); err != nil {
        log.Fatalf("user service error: %v\n", err)
    }
}
```

## 开发环境部署

开发时，可以直接启动`go build`编译的结果。

又由于项目中使用环境变量来控制输入数据，可以使用Makefile来完成：
+ 定义环境变量
+ protoc 编译
+ go build
+ 执行可执行结果

## 产品环境部署

产品环境需要使用Docker运行。

### 独立启动

每个微服务都有独立的Dockerfile和Makefile。在更新时，实现：
+ protoc 编译
+ go build
+ docker build
+ docker run

### 统一启动

进一步可以使用docker-compose管理，一次性启动所有微服务。
