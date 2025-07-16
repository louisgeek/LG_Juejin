# Android NSD 网络服务发现
- Network Service Discovery 网络服务发现是一种基于 DNS-SD（​mDNS）协议的本地网络服务发现机制，用于在 LAN 局域网内注册、发现和连接网络服务（比如打印机、摄像头、文件共享传输、智能家居控制和多人联机游戏等），设备无需手动配置 IP 地址或端口，无需依赖中央服务器（去中心化，通过多播 DNS 实现设备间的零配置发现和解析），设备自行发布和解析主机名称（设备上线时自动广播主机名和 IP 等消息，下线时发送注销消息），允许应用通过指定服务类型和名称来请求服务 
- DNS：域名系统（Domain Name System），用于将域名解析为 IP 地址，通常使用 UDP 单播（端口 53），也可通过 TCP 处理大报文，传统的 DNS 是一种基于 UDP 单播的集中式服务协议，依赖于特定的 DNS 中央服务器，需要配置和维护 DNS 中央服务器
- mDNS：多播 DNS（Multicast DNS），扩展自 DNS 协议（可以理解为 DNS 在本地网络下的轻量化扩展，支持名字解析，mDNS 是 DNS 的补充），通常使用 UDP 组播（端口 5353），专为局域网设计（仅解析以 .local 结尾的域名，可以理解为局域网内的 DNS 系统），用于局域网内设备的名称解析和自动服务发现，mDNS 是一种去中心化的服务发现协议，使用 UDP 组播而不是 UDP 单播，实现设备自动注册和发现，无需依赖中央服务器
- DNS-SD：DNS 服务发现（DNS-Based Service Discovery）是一种基于 DNS 协议扩展（扩展 DNS 记录类型：PTR、SRV 和 TXT，用于描述服务元数据，与 mDNS 兼容）的网络服务发现机制，DNS-SD 通常与 mDNS 结合使用，mDNS 负责局域网内主机名称解析（.local 域），DNS-SD 提供服务发现能力，而无需依赖传统 DNS 中央服务器（不过 DNS-SD 也支持运行在传统的 DNS 上）
- Bonjour：是苹果公司开发的一种零配置网络协议（Zeroconf），用于局域网内设备与服务的自动发现与连接，无需手动配置 IP 地址或端口，本质整合了 mDNS 和 DNS-SD 协议，通过扩展 DNS 记录类型（比如 PTR、SRV 和 TXT）来实现网络服务注册和发现，提供跨平台 API，简化设备互联，Bonjour 看作是具体实现（也是 Zeroconf 最广泛使用的一个实现），DNS-SD 看作是协议框架（定义服务发现的逻辑标准规范），mDNS 看作是底层技术（实现局域网内的零配置解析）
- Avahi：一个开源的 mDNS 和 DNS-SD 实现，常用于 Linux 系统中

## 服务注册
- 设备可以在局域网内注册自己提供的服务
```java
private static final String SERVICE_TYPE = "_http._tcp.";
private NsdManager.RegistrationListener registrationListener;
private void registerService(Context context) {
    int port;
    ServerSocket serverSocket;
    try {
        //0 代表动态分配可用端口
        serverSocket = new ServerSocket(0);
        port = serverSocket.getLocalPort();
        serverSocket.close();
    } catch (IOException e) {
        e.printStackTrace();
    }
    //创建服务信息
    NsdServiceInfo nsdServiceInfo = new NsdServiceInfo();
    nsdServiceInfo.setServiceName("MyNsdAndroidServer"); //服务名称
    nsdServiceInfo.setServiceType(SERVICE_TYPE); //服务类型（DNS-SD 规范）
    nsdServiceInfo.setPort(port);
    nsdServiceInfo.setAttribute("deviceType", "Android"); //自定义属性
    registrationListener = new NsdManager.RegistrationListener() {
        @Override
        public void onRegistrationFailed(NsdServiceInfo serviceInfo, int errorCode) {
            Log.e(TAG, "服务注册失败，错误码：" + errorCode);
        }

        @Override
        public void onUnregistrationFailed(NsdServiceInfo serviceInfo, int errorCode) {

        }

        @Override
        public void onServiceRegistered(NsdServiceInfo serviceInfo) {
            String serviceName = serviceInfo.getServiceName(); //只能读取到 serviceName
            Log.d(TAG, "服务注册成功，服务名称：" + serviceName);
        }

        @Override
        public void onServiceUnregistered(NsdServiceInfo serviceInfo) {

        }
    };
    NsdManager nsdManager = (NsdManager) context.getSystemService(Context.NSD_SERVICE);
    //服务注册
    nsdManager.registerService(nsdServiceInfo, NsdManager.PROTOCOL_DNS_SD, registrationListener);
}

public void unregisterService(Context context) {
    NsdManager nsdManager = (NsdManager) context.getSystemService(Context.NSD_SERVICE);
    //
    if (registrationListener != null) {
        //服务反注册
        nsdManager.unregisterService(registrationListener);
    }
}
```

## 服务发现
- 设备可以在局域网内发现其他设备提供的服务
```java
private static final String SERVICE_TYPE = "_http._tcp.";
private NsdManager.DiscoveryListener discoveryListener;
private void discoverServices(Context context) {
    NsdManager nsdManager = (NsdManager) context.getSystemService(Context.NSD_SERVICE);
    //
    discoveryListener = new NsdManager.DiscoveryListener() {
        @Override
        public void onStartDiscoveryFailed(String serviceType, int errorCode) {
            nsdManager.stopServiceDiscovery(this);
        }
        @Override
        public void onStopDiscoveryFailed(String serviceType, int errorCode) {
            nsdManager.stopServiceDiscovery(this);
        }
        @Override
        public void onDiscoveryStarted(String serviceType) {

        }
        @Override
        public void onDiscoveryStopped(String serviceType) {

        }
        @Override
        public void onServiceFound(NsdServiceInfo serviceInfo) {
            //只能读取到 serviceName 和 serviceType
            String serviceName = serviceInfo.getServiceName();
            String serviceType = serviceInfo.getServiceType();
            if (!SERVICE_TYPE.equals(serviceType)) {
                Log.e(TAG, "onServiceFound: serviceType："+serviceType);
                return;
            }
            if ("MyNsdAndroidServer".equals(serviceName)) {
                    Log.d(TAG, "onServiceFound: Same machine(Same IP): " + serviceName);
                return;
            }
            if (serviceName.contains("MyNsdAndroidServer")){
                //后续的服务解析
                resolveService(context, serviceInfo);
            }
        }
        @Override
        public void onServiceLost(NsdServiceInfo serviceInfo) {

        }
    };
    //服务发现
    nsdManager.discoverServices(SERVICE_TYPE, NsdManager.PROTOCOL_DNS_SD, discoveryListener);
}

public void stopServiceDiscovery(Context context) {
    NsdManager nsdManager = (NsdManager) context.getSystemService(Context.NSD_SERVICE);
    //
    if (discoveryListener != null) {
        //服务发现停止
        nsdManager.stopServiceDiscovery(discoveryListener);
    }
}
```


## 服务解析
- 设备可以解析发现的服务，获取服务的详细信息，解析出具体的 IP 地址和端口号
```java
private void resolveService(Context context, NsdServiceInfo serviceInfo) {
    resolveListener = new NsdManager.ResolveListener() {
        @Override
        public void onResolveFailed(NsdServiceInfo serviceInfo, int errorCode) {

        }

        @Override
        public void onServiceResolved(NsdServiceInfo serviceInfo) {
            //能读取到 serviceName、serviceType、host 和 port
            String host = serviceInfo.getHost().getHostAddress();
            int port = serviceInfo.getPort();
            Log.d(TAG, "服务解析成功，IP 地址：" + host + "，端口：" + port);
        }
    };
    NsdManager nsdManager = (NsdManager) context.getSystemService(Context.NSD_SERVICE);
    //服务解析
    nsdManager.resolveService(serviceInfo, resolveListener);
}
```