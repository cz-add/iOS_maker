> **问题**
> 
> 在手机应用的开发中，通常会将复杂的业务逻辑层实现放在服务端，客户端仅负责表现层。但是对于某些手机应用而言，业务逻辑的实现位于服务端反而是不安全的或是不合理的，而是需要将其逻辑直接在手机端实现。
> 
> **目的**
> 
> 面对不同系统的手机客户端，单独重复实现相同的业务逻辑，并非最佳实践。如何通过第三方语言 Go 语言将业务逻辑封装成库的形式，并以静态打包的方式提供给不同系统的手机客户端使用，是本次调研的目的。

理想目标图：

![输入图片说明](https://images.gitee.com/uploads/images/2021/0602/154959_a1e6feb6_9027123.png "rpc.png")


具体调研内容包括:

*   [ ] iOS 应用实现 gRPC 调用
*   [ ] Android 应用实现 gRPC 调用
*   [ ] GoMobile SDK 在 iOS & Android 上的集成
*   [ ] GoMobile SDK 在 iOS & Android 上的边界
*   [ ] C/S 架构 or 静态库

其中关于 gRPC 在 iOS 与 Android 的实现，本身官方就已经提供了样例。本次调研会用到相关内容，所以将其作为调研的一部分记录下来，方便后来者阅读。

## 1\. 环境安装

其实很多东西没那么难，只是需要开始而已。 为了完成目标调研，开始第一部分的调研工作。以文字形式记录下来，方便后来者。

### 1.1 XCode 安装

没什么好说的，直接 AppStore 下载安装。有点慢，一边下载一边准备其它环境。

### 1.2 Cocoapod 安装

类似与其它语言的第三方库管理工具。也没什么好说的，登录官网，按说明安装。

```
$: sudo gem install cocoapods
```

### 1.3 protoc 命令安装

因为 gRPC 的广泛使用， ProtoBuf 协议被广泛用于字节编码与解码的协议， 其具体指南参考[官网]()。话不多说，安装：

```
$: curl -LOk https://github.com/protocolbuffers/protobuf/releases/download/v3.5.1/protoc-3.9.0-rc-1-osx-x86_64.zip
$: unzip protoc-3.9.0-rc-1-osx-x86_64.zip -d proto_buffer && cd proto_buffer
$: sudo cp bin/protoc /usr/local/bin
$: sudo cp -R include/google/protobuf/ /usr/local/include/google/protobuf
$: protoc --version
```

### 1.4 protoc 插件安装

protoc 主要是通过解析 `.proto` 格式的文件, 再根据具体插件生成相应语言代码。
考虑到需要同时实现客户端与服务端的代码，所以必须安装以下三个插件：

*   swift
*   swiftgrpc
*   go 主要生成 go 代码， 用于服务端实现

swift 插件安装：

```
$: git clone https://github.com/grpc/grpc-swift.git
$: cd grpc-swift
$: git checkout tags/0.5.1
$: make
$: sudo cp protoc-gen-swift protoc-gen-swiftgrpc /usr/local/bin
```

go 插件安装：

前提是需要安装 Go 语言的开发环境， 可参考官网。`protoc-gen-go`安装详细[指南](https://github.com/golang/protobuf).

```
$: go get -u github.com/golang/protobuf/protoc-gen-go
```

## 2 定义 proto 接口

既然是最简单的调研，就用最简单的 Hello 服务。创建项目路径并定义：

```
$: mkdir grpc-apps
$: cd grpc-apps
$: mkdir proto
$: cat <<EOF > proto/hello.proto
syntax = "proto3";

option java_multiple_files = true;
option java_package = "com.gitdig.helloworld";
option java_outer_classname = "HelloWorldProto";

package helloworld;

service Greeter {
  rpc SayHello (HelloRequest) returns (HelloReply) {}
}

message HelloRequest {
  string name = 1;
}

message HelloReply {
  string message = 1;
}
EOF
```

## 3\. 服务端实现

在项目目录中创建服务端目录与proto生成目录，同时编写一个简单的服务端：

```
$: cd grpc-apps
$: mkdir go go/client go/server go/hello
# 生成 Go 代码到 go/hello 文件夹
$: protoc -I proto proto/hello.proto --go_out=plugins=grpc:./go/hello/
```

分别编辑 Go 版本 client 与 server 实现。确认服务正常运行。

### 3.1 Go 服务端

编辑 `server/server.go` 文件：

```
package main

import (
    pb "github.com/liujianping/grpc-apps/go/helloworld"
)

import (
    "context"
    "fmt"
    "log"
    "net"

    "google.golang.org/grpc"
)

type HelloServer struct{}

// SayHello says 'hi' to the user.
func (hs *HelloServer) SayHello(ctx context.Context, req *pb.HelloRequest) (*pb.HelloReply, error) {
    // create response
    res := &pb.HelloReply{
        Message: fmt.Sprintf("hello %s from go", req.Name),
    }

    return res, nil
}

func main() {
    var err error

    // create socket listener
    l, err := net.Listen("tcp", ":50051")
    if err != nil {
        log.Fatalf("error: %v\n", err)
    }

    // create server
    helloServer := &HelloServer{}

    // register server with grpc
    s := grpc.NewServer()
    pb.RegisterGreeterServer(s, helloServer)

    log.Println("server serving at: :50051")
    // run
    s.Serve(l)
}

```

运行服务端程序:

```
$: cd grpc-apps/go
$: go run server/server.go
2019/07/03 20:31:06 server serving at: :50051
```

### 3.2 Go 客户端

编辑 `client/client.go` 文件：

```
package main

import (
    pb "github.com/liujianping/grpc-apps/go/helloworld"
)

import (
    "context"
    "fmt"
    "log"

    "google.golang.org/grpc"
)

func main() {
    var err error

    // connect to server
    conn, err := grpc.Dial("localhost:50051", grpc.WithInsecure())
    if err != nil {
        log.Fatalf("error: %v\n", err)
    }
    defer conn.Close()

    // create client
    client := pb.NewGreeterClient(conn)

    // create request
    req := &pb.HelloRequest{Name: "JayL"}

    // call method
    res, err := client.SayHello(context.Background(), req)
    if err != nil {
        log.Fatalf("error: %v\n", err)
    }

    // handle response
    fmt.Printf("Received: \"%s\"\n", res.Message)
}

```

执行客户端程序：

```
$: cd grpc-apps/go
$: go run client/client.go
Received: "hello JayL from go"
```

Go 客户端/服务端通信成功。

## 4\. iOS 项目

### 4.1 创建一个最简单的单视图项目

创建一个名为 iosDemo 的单视图项目，选择 swift 语言， 存储路径放在 `grpc-apps` 下。完成创建后，正常运行，退出程序。

### 4.2 初始化项目 Pod

在命令行执行初始化:

```
$: cd grpc-apps/iosDemo
# 初始化
$: pod init
$: vim Podfile
```

编辑 Podfile 如下:

```
# Uncomment the next line to define a global platform for your project
# platform :ios, '9.0'

target 'iosDemo' do
  # Comment the next line if you don't want to use dynamic frameworks
  use_frameworks!

  # Pods for iosDemo
  pod 'SwiftGRPC'
end
```

完成编辑后保存，执行安装命令:

```
$: pod install
```

安装完成后，项目目录发生以下变更：

```
$: git status
On branch master
Changes not staged for commit:
  (use "git add <file>..." to update what will be committed)
  (use "git checkout -- <file>..." to discard changes in working directory)

    modified:   iosDemo.xcodeproj/project.pbxproj

Untracked files:
  (use "git add <file>..." to include in what will be committed)

    Podfile
    Podfile.lock
    Pods/
    iosDemo.xcworkspace/

no changes added to commit (use "git add" and/or "git commit -a")
```

通过命令行 `open iosDemo.xcworkspace` 打开项目，对项目中的info.list的以下设置进行修改：

![](https://upload-images.jianshu.io/upload_images/24396273-bc291910462ca980.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


通过设置，开启非安全的HTTP访问方式。

### 4.3 生成 gRPC swift 代码

类似 Go 代码生成，现在生成 swift 代码：

```
$: cd grpc-apps
# 创建生成文件存放目录
$: mkdir swift
# 生成 swift 文件
$: protoc -I proto proto/hello.proto \
    --swift_out=./swift/ \
    --swiftgrpc_out=Client=true,Server=false:./swift/
# 生成文件查看
$: tree swift
swift
├── hello.grpc.swift
└── hello.pb.swift
```

### 4.4 将生成代码集成到 iOS 项目

XCode中添加生成代码需要通过拖拽的方式，对于后端开发而言，确实有点不可理喻。不过既然必须这样就按照规则：

现在在 iOS 的视图加载函数增加 gRPC 调用过程:

```
class ViewController: UIViewController {

    override func viewDidLoad() {
        super.viewDidLoad()
        // Do any additional setup after loading the view.
        let client = Helloworld_GreeterServiceClient(address: ":50051", secure: false)
        var req = Helloworld_HelloRequest()
        req.name = "JayL"
        do {
            let resp = try client.sayHello(req)
            print("resp: \(resp.message)")
        } catch {
            print("error: \(error.localizedDescription)")
        }
    }
}
```

查看日志输出`resp: hello iOS from go`, iOS 应用调用 gRPC 服务成功。
