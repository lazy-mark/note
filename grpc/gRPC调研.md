# gRPC调研

参考链接：https://zhuanlan.zhihu.com/p/85151508

## gRPC是什么？

字面意思：远程过程调用。

- gRPC是Google开发的高性能、通用的开源RPC框架，主要面向移动应用开发。
- 基于HTTP/2协议标准而设计，基于Protobuf序列化协议开发。
- 支持众多语言。
- 在gRPC中一个客户端可以像本地对象那样直接调用位于不同机器上的服务应用的方法。



和其它gpc类似，gRPC也是基于定义一个服务，指定服务可以被远程调用的方法以及他们的参数和返回类型。在服务端，实现服务的接口然后运行一个gRPC服务来处理可出端的请求。在客户端，客户端拥有一个存根（stub），提供与服务器相同的方法。



### HTTP 2

- 二进制框架，高性能，传输更轻便，解码更安全。
- 使用HPACK压缩标头，减少管理费用并提高性能。
- 多路复用，客户端和服务器可以并行发送多个请求和响应，提高网络利用率。
- 允许服务器推送，在客户端发出单个请求的情况下，服务器可以发回多个响应，可减少客户端和服务器之间的往返延迟。当服务器确切知道客户端将需要哪些资源时，并在他们被要求之前发送。



与HTTP1.1比较

|         传输协议         | HTTP/2 | HTTP/1.1 |
| :----------------------: | :----: | :------: |
|         传输格式         | 二进制 |   文本   |
|           头部           |  压缩  |  纯文本  |
|         多路复用         |  支持  |  不支持  |
| 单个连接发送请求或响应数 |  多个  |    1     |
|         服务推送         |  支持  |  不支持  |



HTTP2如何工作？

- 单个TCP连接可承载多个双向流，每个流都有一个唯一的标识符，并携带多个双向消息。
- 每个消息（请求或响应）都可以分解为多个二进制帧。帧是承载不同类型数据的最小单位，例如：标头、设置、优先级、数据等。



## protobuf协议缓冲区

 默认情况下，gRPC使用protocol buffer，用于序列化结构化数据（可与JSON一起使用）。使用协议缓冲区的第一步是在proto文件中为要序列化的数据定义结构。

`.proto` 中可定义消息，其中每个消息都是消息的逻辑记录。定义数据结构后，就可以使用protocol buffer编译器生成所选语言的数据访问类。访问类为每个字段提供了简单的访问器，以及将整个结构序列化为原始字节或原始字节中解析出整个结构的方法。

`.proto` 文件中还能定义gRPC服务，并将gRPC方法参数和返回类型指定为protocol buffer消息。



为什么使用protobuf？

- 易读、自动生成多语言。
- 二进制表示数据，比JSON或XML等文本的格式更有效地进行序列化。
- 提供了客户端和服务器之间的强类型API，安全。



### gRPC的类型

- 一元rpc
  - 客户端发送1条单个请求消息的位置，服务器回复一个响应。
- 服务器流式rpc
  - 客户端仅发送1条请求消息，服务器回复多个响应流。
- 客户端流式rpc
  - 客户端将发送多个消息流，并且期望服务器仅发送回1个响应。
- 双向流式rpc
  - 客户端和服务器并行接受和发送多个消息。



|     特性     |      GRPC       |      REST       |
| :----------: | :-------------: | :-------------: |
|     协议     |     HTTP/2      |    HTTP/1.1     |
| 有效负载数据 | protocel buffer |      JSON       |
|   API严格    |      严格       |      宽松       |
|   代码生成   |   内置在gRPC    | swagger/OpenAPI |
|    安全性    |     TLS/SSL     |     TLS/SSL     |
|      流      |     双向的      |       1路       |
| 浏览器支持度 |   不容易支持    |      支持       |



使用gRPC的好处

- 低延迟、高性能
- 强制API合同
- 支持多语言
- 点对点通信，提供双向流提供了出色的支持。



### 服务端创建流程



### 客户端创建流程



### 异常处理

方法一：设置异常 message 到Status的 description。

消息定义：

```protobuf
syntax = "proto3";

option java_multiple_files = true;
// 消息定义存放的包结构
option java_package = "io.grpc.test";
option java_outer_classname = "Proto3";
option objc_class_prefix = "xxx";
option java_string_check_utf8 = false;

package test;

service OrderService {
  rpc testException(OrderRequest) returns (OrderResponse){}
}

message OrderRequest {
  int32 page = 1;
  int32 limit = 2;
  string orderId = 34;
  int64 createdTime = 50;
}

message OrderResponse {
  int32 size = 1;
  OrderGrpc order = 2;
}
```





服务端：

```java
@Override
public void testException(OrderRequest request, 
                          StreamObserver<OrderResponse> responseObserver) {
    try {
        if (request.getOrderId().equals("error")) {
            throw new CommonException("测试异常");
        }
    } catch (Exception e) {
        responseObserver.onError(Status.INVALID_ARGUMENT
                                 .withCause(e)
                                 .withDescription(e.getMessage())
                                 .asRuntimeException()
                                );
    }
}
```

使用Status.INVALID_ARGUMENT指定异常的code，这个code表示参数有问题，正常服务端需要明确抛出的异常大多数是参数问题，如果是服务问题，不用做特殊处理，让它直接抛出。

在withDescription方法中自定义我们的异常信息，抛出到客户端。



客户端：

```java
public void testException() {
    try {
        blockingStub.testException(OrderRequest.newBuilder().setOrderId("error").build());
    } catch (StatusRuntimeException e) {
        System.out.println(e.getStatus());
        System.out.println(e.getMessage());
        if (e.getStatus().getCode() == Status.Code.INVALID_ARGUMENT) {
            System.out.println(e.getStatus().getDescription());
            // 返回到前端
            throw new RuntimeException(e.getStatus().getDescription());
        } else {
            throw e;
        }
    }
}
```

自定义的异常信息通过 `e.getStatus().getDescription()` 获取。



方法二：通过MetaData传递更详细的错误信息。

在proto文件中自定义一个消息ErrorDesc。

```protobuf
message ErrorDesc {
	string message = 1;
}
```

在消息中，可定义其它字段，丰富错误的信息。



服务端：

```java
private static final Metadata.Key<ErrorDesc> ERROR_DESC_KEY = 
  ProtoUtils.keyForProto(ErrorDesc.getDefaultInstance());

@Override
public void testMoreException(OrderRequest request, 
                              StreamObserver<OrderResponse> responseObserver) {
    try {
        if (request.getOrderId().equals("error")) {
            throw new RuntimeException("测试多异常信息");
        }
        OrderResponse response = OrderResponse.newBuilder().build();
        responseObserver.onNext(response);
        responseObserver.onCompleted();
    } catch (Exception e) {
        Metadata metadata = new Metadata();
        ErrorDesc.Builder build = ErrorDesc.newBuilder().setCode(500)
            .setMessage(e.getMessage());
        metadata.put(ERROR_DESC_KEY, build.build());
        responseObserver.onError(Status.INVALID_ARGUMENT
                                 .withCause(e)
                                 .asRuntimeException(metadata));
    }
}
```

通过  `asRuntimeException`  将异常信息传输到客户端。



客户端：

```java
private static final Metadata.Key<ErrorDesc> ERROR_DESC_KEY = 	
  ProtoUtils.keyForProto(ErrorDesc.getDefaultInstance());

public void testMoreException() {
    try {
        OrderResponse response =
            blockingStub.testMoreException(OrderRequest
                                           .newBuilder()
                                           .setOrderId("error")
                                           .build());
        System.out.println(response.getOrder());
    } catch (StatusRuntimeException e) {
        if (Status.Code.INVALID_ARGUMENT == e.getStatus().getCode()) {
            Metadata metadata = Status.trailersFromThrowable(e);
            ErrorDesc desc = metadata.get(ERROR_DESC_KEY);
            if (desc != null) {
                System.out.println(desc.getCode() + ":" + desc.getMessage());
            }
        }
    }
}
```

客户端中也需要定义  `Metadata.Key`  ，否则无法获取到异常信息。



异步调用的异常处理：

服务端不变，客户端：

```java
public void testFutureException() {
    try {
        ListenableFuture<OrderResponse> listenableFuture =
            futureStub.testException(OrderRequest.newBuilder().setOrderId("error").build());
        System.out.println(listenableFuture.get().getOrder());
    } catch (InterruptedException e) {
        System.out.println(e.getMessage());
    } catch (ExecutionException e) {
        System.out.println(e.getMessage());
        if (e.getCause() instanceof StatusRuntimeException) {
            StatusRuntimeException cause = (StatusRuntimeException) e.getCause();
            if (Status.Code.INVALID_ARGUMENT == cause.getStatus().getCode()) {
                System.out.println("异常信息:" + cause.getStatus().getDescription());
                throw new RuntimeException(cause.getStatus().getDescription());
            }
        }
    }
}
```



未知异常：

服务端：

在proto文件中添加服务方法 `testUnkownException`。



```java
@Override
public void testUnkownException(OrderRequest request, 
                                StreamObserver<OrderResponse> responseObserver) {
    throw new RuntimeException("未知异常");
}
```

客户端：

```java
public void testUnkonwException() {
    try {
        OrderResponse response = blockingStub.testUnkownException(OrderRequest.newBuilder().build());
        System.out.println(response);
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```

此时客户端并不能直接获取到服务端的异常信息。

```
io.grpc.StatusRuntimeException: UNKNOWN
	at io.grpc.stub.ClientCalls.toStatusRuntimeException(ClientCalls.java:221)
	at io.grpc.stub.ClientCalls.getUnchecked(ClientCalls.java:202)
	at io.grpc.stub.ClientCalls.blockingUnaryCall(ClientCalls.java:131)
	at io.grpc.test.OrderServiceGrpc$OrderServiceBlockingStub.testUnkownException(OrderServiceGrpc.java:287)
	at com.xxx.grpc.Client.testUnkonwException(Client.java:92)
	at com.xxx.grpc.Client.main(Client.java:106)
```





错误状态码：

一般错误

|            状态码             |                例子                |
| :---------------------------: | :--------------------------------: |
|     GRPC_STATUS_CANCELLED     |         客户端申请取消请求         |
| GRPC_STATUS_DEADLINE_EXCEEDED | 服务器返回状态之前的截止时间已过期 |
|   GRPC_STATUS_UNIMPLEMENTED   |          服务器找不到方法          |
|    GRPC_STATUS_UNAVAILABLE    |             服务器关闭             |
|      GRPC_STATUS_UNKNOWN      |          服务器引发了异常          |



网络故障

|            状态码             |                   例子                    |
| :---------------------------: | :---------------------------------------: |
| GRPC_STATUS_DEADLINE_EXCEEDED | 截止日期到期前未传输任何数据/未发生故障。 |
|    GRPC_STATUS_UNAVAILABLE    |        连接断开之前传输的一些数据         |
|                               |                                           |



协议故障

|             状态码             |                例子                |
| :----------------------------: | :--------------------------------: |
|      GRPC_STATUS_INTERNAL      |         客户端申请取消请求         |
|   GRPC_STATUS_UNIMPLEMENTED    | 服务器返回状态之前的截止时间已过期 |
| GRPC_STATUS_RESOURCE_EXHAUSTED |          服务器找不到方法          |
|      GRPC_STATUS_INTERNAL      |          违反流量控制协议          |
|      GRPC_STATUS_UNKNOWN       |          服务器引发了异常          |
|      GRPC_STATUS_INTERNAL      |    错误解析请求/响应协议缓存区     |
|  GRPC_STATUS_UNAUTHENTICATED   |    未经身份验证，无法获取元数据    |





## gRPC拦截器

### 实现Jwt认证

gRPC的SSL/TLS安全机制，设置过程比较复杂，比如：证书签名，需要在客户端、服务端设置。用Jwt会更加便捷，更加安全和功能强大，因为除Jwt的加密签名之外还可以把私密的用户信息放在Jwt里加密后在客户端和服务端之间传递。当然，最基本的是通过Jwt的验证机制可以控制客户端对某些功能的使用权限。

下面开始基于Jwt的身份验证保护gRPC服务，首先引入Jwt相关依赖。

```xml
<dependency>
  <groupId>io.jsonwebtoken</groupId>
  <artifactId>jjwt</artifactId>
  <version>0.9.1</version>
</dependency>
<!--解决java.lang.NoClassDefFoundError: javax/xml/bind/DatatypeConverter-->
<dependency>
  <groupId>javax.xml.bind</groupId>
  <artifactId>jaxb-api</artifactId>
  <version>2.3.0</version>
</dependency>
```

引入公共类`Constant`：

```java
public final class Constant {
  public static String JWT_SIGNING_KEY = "L8hHXsaQOUjk5rg7XPGv4eL36anlCrkMz8CJ0i/8E/0=";
  public static final String BEARER_TYPE = "Bearer";
  public static final Metadata.Key<String> AUTHORIZATION_METADATA_KEY = 
    Metadata.Key.of("Authorization", ASCII_STRING_MARSHALLER);
  public static final Context.Key<String> CLIENT_ID_CONTEXT_KEY = Context.key("clientId");
  private Constant() {}
}
```

protobuf文件

```protobuf
syntax = "proto3";
service OrderService {
  rpc addOrder(OrderGrpc) returns (OrderResponse) {}
}
// 请求结构体
message OrderGrpc {
  string orderId = 1;
  string orderName = 2;
}
// 返回结构体
message OrderResponse {
  int32 size = 1;
  OrderGrpc order = 2;
}
```

服务端实现OrderService，

```java
public class OrderGrpcServiceImpl extends OrderServiceGrpc.OrderServiceImplBase {
  private static final Map<String, OrderGrpc> data = new HashMap<>();
  @Override
  public void addOrder(OrderGrpc request, StreamObserver<OrderResponse> responseObserver) {
      data.put(request.getOrderId(), request);
      OrderResponse response = OrderResponse.newBuilder().setOrder(request).setSize(1).build();
      responseObserver.onNext(response);
      responseObserver.onCompleted();
  }
}
```

服务端启动类，将实现类 `OrderGrpcServiceImpl` 加入到gRPC中。

```java
public class Server {

  private io.grpc.Server server;


  private void start() throws IOException {
      this.server = ServerBuilder.forPort(7777).addService(new OrderGrpcServiceImpl()).build().start();
      System.out.println("服务端启动.....");
      Runtime.getRuntime().addShutdownHook(new Thread(() -> {
          System.err.println("------shutting down gRPC server since JVM is shutting down-------");
          Server.this.stop();
          System.err.println("------server shut down------");
      }));
  }

  private void stop() {
      if (server != null) {
          server.shutdown();
      }
  }

  private void blockUntilShutdown() {
      if (server != null) {
          try {
              server.awaitTermination();
          } catch (InterruptedException e) {
              System.err.println("program error!");
          }
      }
  }

  public static void main(String[] args) throws IOException {
      Server server = new Server();
      server.start();

      server.blockUntilShutdown();
  }
}
```

客户端访问

```java
public class Client {

    private final ManagedChannel channel;
    private final OrderServiceGrpc.OrderServiceBlockingStub blockingStub;
    private final OrderServiceGrpc.OrderServiceFutureStub futureStub;

    public Client() {
        this.channel = ManagedChannelBuilder.forAddress("127.0.0.1", 7777).usePlaintext().build();
        blockingStub = OrderServiceGrpc.newBlockingStub(channel);
        futureStub = OrderServiceGrpc.newFutureStub(channel);
    }

    public void addOrder() {
        OrderGrpc request = OrderGrpc.newBuilder().setOrderId("1").setOrderName("Mac").build();
        OrderResponse orderResponse = blockingStub.withCallCredentials(new JwtCredential("12345678")).addOrder(request);
        System.out.println(orderResponse);
    }

    public static void main(String[] args) {
        Client client = new Client();
        System.out.println("==============add order========================");
        client.addOrder();
        client.channel.shutdown();
    }
}
```

此时运行后，客户端就可以打印了服务端返回的消息。

```reStructuredText
==============add order========================
size: 1
order {
  orderId: "1"
  orderName: "Mac"
}
```

到这就实现了gRPC的调用，下面开始集成Jwt，首先在服务端添加拦截器。

```java
public class JwtServerInterceptor implements ServerInterceptor {

    private JwtParser parser = Jwts.parser().setSigningKey(Constant.JWT_SIGNING_KEY);

    @Override
    public <ReqT, RespT> ServerCall.Listener<ReqT> interceptCall(ServerCall<ReqT, RespT> call, Metadata headers,
        ServerCallHandler<ReqT, RespT> next) {
        String value = headers.get(Constant.AUTHORIZATION_METADATA_KEY);
        Status status = Status.OK;
        if (value == null) {
            status = Status.UNAUTHENTICATED.withDescription("Authentication token is missing.");
        } else if (!value.startsWith(Constant.BEARER_TYPE)) {
            status = Status.UNAUTHENTICATED.withDescription("Unknown Authentication type");
        } else {
            Jws<Claims> claims = null;
            String token = value.substring(Constant.BEARER_TYPE.length()).trim();
            try {
                claims = parser.parseClaimsJws(token);
            } catch (JwtException e) {
                status = Status.UNAUTHENTICATED.withDescription(e.getMessage()).withCause(e);
            }
            if (claims != null) {
                Context ctx =
                    Context.current().withValue(Constant.CLIENT_ID_CONTEXT_KEY, claims.getBody().getSubject());
                return Contexts.interceptCall(ctx, call, headers, next);
            }
        }
        call.close(status, new Metadata());
        return new ServerCall.Listener<ReqT>() {};
    }
}
```

在服务端启动类注册我们写的拦截器实现。

```java
public class Server {
 //...
 private void start() throws IOException {
        this.server = ServerBuilder.forPort(7777).intercept(new JwtServerInterceptor()).addService(new OrderGrpcServiceImpl()).build().start();
        System.out.println("服务端启动.....");
        Runtime.getRuntime().addShutdownHook(new Thread(() -> {
            System.err.println("------shutting down gRPC server since JVM is shutting down-------");
            Server.this.stop();
            System.err.println("------server shut down------");
        }));
    }
  
  //...
}
```

此时，如果客户端不传递凭证，就会被我们自定义的拦截器拦截，抛出异常。

```reStructuredText
==============add order========================
Exception in thread "main" io.grpc.StatusRuntimeException: UNAUTHENTICATED: Authentication token is missing.
	at io.grpc.stub.ClientCalls.toStatusRuntimeException(ClientCalls.java:221)
	at io.grpc.stub.ClientCalls.getUnchecked(ClientCalls.java:202)
	at io.grpc.stub.ClientCalls.blockingUnaryCall(ClientCalls.java:131)
	at io.grpc.test.OrderServiceGrpc$OrderServiceBlockingStub.addOrder(OrderServiceGrpc.java:259)
	at com.zsj.grpc.Client.addOrder(Client.java:27)
	at com.zsj.grpc.Client.main(Client.java:102)
```

因此，客户端也需要进行改造，新建一个 `JwtCredential `类携带凭证数据的一个类，在gRPC中提供了 `CallCredentials` 接口，我们实现该接口即可。

```java
public class JwtCredential implements CallCredentials {

    private final String subject;

    public JwtCredential(String subject) {
        this.subject = subject;
    }

    @Override
    public void applyRequestMetadata(MethodDescriptor<?, ?> method, Attributes attrs, Executor executor,
        MetadataApplier applier) {
        try {
            final String jwt =
                Jwts.builder().setSubject(subject).signWith(SignatureAlgorithm.HS256, Constant.JWT_SIGNING_KEY).compact();
            executor.execute(() -> {
                Metadata header = new Metadata();
                header.put(Constant.AUTHORIZATION_METADATA_KEY, String.format("%s %s", Constant.BEARER_TYPE, jwt));
                applier.apply(header);
            });
        } catch (Throwable e) {
            applier.fail(Status.UNAUTHENTICATED.withCause(e));
        }
    }

    @Override
    public void thisUsesUnstableApi() {
        // no loop
    }
}
```

在客户端调用处传递凭证

```java
public void addOrder() {
    OrderGrpc request = OrderGrpc.newBuilder().setOrderId("1").setOrderName("Mac").build();
    OrderResponse orderResponse = blockingStub.withCallCredentials(new JwtCredential("12345678")).addOrder(request);
    System.out.println(orderResponse);
}
```

此时，启动服务端和客户端后，依旧打印了正常的信息。



在gRPC中，利用Jwt来做身份认证，可能也存在问题。

1. 每调用一个gRPC方法，就要进行加密和解密操作，影响效率。



gRPC原理解析：https://blog.csdn.net/iteye_19607/article/details/82651139