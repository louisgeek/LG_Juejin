# Android OkHttp Interceptor
- 采用了责任链模式，拦截器允许在请求和响应的过程中插入自定义逻辑
- Application Interceptor 执行频率更高（无论是否走网络），需要注意避免耗时操作阻塞主线程


## 拦截器执行顺序
- 请求顺序：
    - 所有自定义 Application Interceptor 应用拦截器按添加顺序依次执行（应用拦截器是最先执行的拦截器）
    - 内置拦截器 RetryAndFollowUpInterceptor 重试重定向，处理请求重定向（比如 30X 响应码，重定向时直接发起新的请求）和网络请求失败后进行重试，通过 while 死循环实现多次尝试
    - 内置拦截器 BridgeInterceptor 桥接拦截器，作为应用层与网络层的桥梁，自动为请求添加必要头信息（比如 Content-Type、Cookie、Host 和 User-Agent 等），设置 Connection 支持 Keep-Alive，并移除响应中的临时头，设置 Accept-Encoding 支持 gzip 压缩传输，并且在接收到内容后进行解压
    - 内置拦截器 CacheInterceptor 缓存拦截器，管理 HTTP 缓存逻辑，优先返回缓存响应（倘若命中了缓存，那就意味着不需要往下继续走网络请求了，所以后续就不会执行网络拦截器了），否则触发网络请求并更新缓存
    - 内置拦截器 ConnectInterceptor 连接拦截器，负责与服务器建立 TCP 连接，为后续请求准备网络通道
    - 所有自定义 Network Interceptor 网络拦截器按添加顺序依次执行（网络拦截器位于内置拦截器 ConnectInterceptor 和内置拦截器 CallServerInterceptor 之间）
    - 内置拦截器 CallServerInterceptor 网络请求，拦截器链的最后一环，发送最终请求至服务器并获取原始响应（向服务器发起真正的访问请求，获取 Response）
- 响应顺序：
    - 返回响应给自定义 Network Interceptor 网络拦截器按添加顺序逆序处理
    - 返回响应给自定义 Application Interceptor 应用拦截器按添加顺序逆序处理


## Application Interceptor 应用拦截器
- 位于整个请求链的最前端，在请求被发送到网络之前（可能处理缓存响应，比如命中缓存时直接返回，不触发网络请求）和响应返回到应用程序之后进行拦截（能拦截所有请求，无法获取底层网络连接 Connection 对象）
- 每次请求只调用一次，不处理重定向、重试等等中间过程
- 通过 addInterceptor 添加
- 应用：统一添加公共请求头（比如统一添加 Token）、统一处理响应（比如 401 Token 令牌过期）、全局错误码处理、统一进行请求响应日志打印记录和统一加密解密等

```java
//为所有请求添加 Authorization 头
OkHttpClient okHttpClient = new OkHttpClient.Builder()
    .addInterceptor(new Interceptor() {
        @Override
        public Response intercept(Chain chain) throws IOException {
            Request request = chain.request().newBuilder()
                .addHeader("Accept", "application/json")
                .addHeader("Content-Type", "application/json")
                .addHeader("Authorization", "Bearer token")
                .build();
            return chain.proceed(request);
        }
    })
    .build();
```

```java
//统一处理响应的头信息
OkHttpClient okHttpClient = new OkHttpClient.Builder()
    .addInterceptor(new Interceptor() {
        @Override
        public Response intercept(Chain chain) throws IOException {
            Response response = chain.proceed(chain.request());
            String traceId = response.header("traceId");
            System.out.println("响应 traceId: " + traceId);
            return response;
        }
    })
    .build();
```

```java
//全局错误码处理
public class ErrorHandleInterceptor implements Interceptor {
    @Override
    public Response intercept(Chain chain) throws IOException {
        Request request = chain.request().newBuilder()
            .addHeader("Authorization", "Bearer " + getToken()) //从本地获取 Token
            .build();
        Response response = chain.proceed(request);
        //Response response = chain.proceed(chain.request());
        if (response.code() == 401) {
            //重新获取 Token 并重试请求
            refreshToken();
            Request newRequest = request.newBuilder()
                .header("Authorization", "Bearer " + getToken())
                .build();
            return chain.proceed(newRequest);
        }
        return response;
    }
}
OkHttpClient okHttpClient = new OkHttpClient.Builder()
    .addInterceptor(new ErrorHandleInterceptor())
    .build();
```

```java
//统一进行日志打印记录
HttpLoggingInterceptor httpLoggingInterceptor = new HttpLoggingInterceptor();
httpLoggingInterceptor.setLevel(HttpLoggingInterceptor.Level.BODY);

OkHttpClient okHttpClient = new OkHttpClient.Builder()
    .addInterceptor(httpLoggingInterceptor)
    .build();
```

## Network Interceptor 网络拦截器
- 在请求实际被发送到网络前和响应从网络返回后进行拦截（不处理缓存响应，仅能拦截实际发送到网络的请求，能访问底层网络连接 Connection 对象，获取 TLS 协议、IP 等底层信息）
- 每次网络请求都可能调用（包括重试、重定向）
- 通过 addNetworkInterceptor 添加
- 应用：网络调试（查看网络请求和响应的原始数据）、网络性能监控（监控耗时）和拦截重试（根据网络状态自动重试请求）

```java
class NetworkLoggingInterceptor implements Interceptor {
    @Override
    public Response intercept(Chain chain) throws IOException {
        long startTime = System.nanoTime();
        Request request = chain.request();
        //获取连接信息（仅网络拦截器可用）
        Connection connection = chain.connection();
        if (connection != null) {
            Socket socket = connection.socket();
            String host = socket.getInetAddress().getHostAddress();
            Log.d("Network", "Request url=" + request.url() + " host=" + host);
        }
        Response response = chain.proceed(request);
        long endTime = System.nanoTime();
        Log.d("Network", "diffTime: " + (endTime - startTime) + " ns");
        return response;
    }
}
OkHttpClient okHttpClient = new OkHttpClient.Builder()
    .addNetworkInterceptor(new NetworkLoggingInterceptor())
    .build();
```

## 总结
- Application Interceptor 应用拦截器：用于业务逻辑处理（认证、日志和错误处理等），避免耗时操作
- Network Interceptor 网络拦截器：用于网络层面的处理（压缩、加密和网络监控等），可能被多次调用
- 执行顺序：多个 Application Interceptor 按添加顺序执行，Network Interceptor 在 Application Interceptor 之后执行，正因为应用拦截器在 RetryAndFollowUpInterceptor 之前，所以一旦发生错误重试或网络重定向，网络拦截器可能会执行多次（因为相当于进行了二次请求），而应用拦截器只走了一次