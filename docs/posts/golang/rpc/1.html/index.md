# netrpc源码阅读



# netrpc源码阅读

> [[源码地址]](https://github.com/golang/go/tree/master/src/net/rpc)


## Client.go

核心结构体：

- Call：主要表示调用RPC过程中的信息
- Client：主要负责RPC客户端

- gobClientCodec：主要负责调用RPC具体实现方法

核心接口：

- ClientCodec

重点：

- ClientCodec接口规定了在调用过程最主要的几个具体步骤：`WriteRequest()`、`ReadResponseHeader()`、`ReadResponseBody()`、`Close()`；
- Client封装了RPC客户端未暴露的主要的两个动作即`send()`、`input()`，分别是发送操作和接收操作，利用互斥锁保证线程安全，其循环接收response且若出现错误则终结掉所有Call结构体，需要其中对于Call结构体的处理。而Client暴露的`Go()`和`Call()`就是其客户端库的入口，实质上是同步进行操作而异步进行请求具体方法返回其call信息，且区别主要在于前者多了一个done通道可以进行异步通信操作，`Call()`实质上也是对于`Go()`方法的调用；
- `NewClientWith()`方法即客户端初始化其中包含了`NewClientWithCodec()`，默认以gobClientCodec方式作为RPC调用具体实现方法，且初始化时协程开启进行`input()`函数进行接收；
- `DailHTTP()`方法和`Dial()`方法即客户端网络处理，前者包含了`DialHTTPPath()`方法即以HTTP协议进行通信，且默认为HTTP1.0协议，后者可以连接其特殊网络地址；

## Server.go

核心结构体：

- Request/Response：主要负责请求返回涉及rpc的信息
- Server：主要负责RPC服务端
- gobServerCodec：主要负责调用RPC具体实现方法
- methodType/service：主要表示调用方法和注册服务的信息

核心接口：

- ServerCodec

重点：

- `isExportedOrBuiltinType()`方法判断类型是否为导出还是内置的，反射方法调用中需要保证其类型和方法为可导出的；
- Server封装了其服务端主要的方法，涉及到`Register()`、`ServerConn()`、`ServerCode()`、`ServerRequest()`、`ServerResponse()`、`Accepct()`、`ServerHTTP()`等；
- `Register()`方法及`RegisterName()`方法都是主要作用为注册服务，其中注册的服务中的参数格式和参数类型都有相关要求，涉及到参数个数、参数类型、参数是否可导出等，主要是利用反射获取相关信息进行判断；
- `ServeCodec()`方法和`ServeRequest()`方法就是接收处理请求，将得到的信息进行相应的Call操作，前者为循环接收及异步调用，且保证阻塞直至所有Call调用完成，后者则为一次性调用且为同步调用；
- 请求中的信息需要先进行解码后，经过处理后再编码发送其返回信息，其中`call()`就是处理过程中的核心方法，涉及invoke服务方法并编码进行返回；
- `ServeHTTP()`方法处理其服务端的网络连接，注册初始化服务端连接相关信息，进行HTTP1.0协议的通信；
