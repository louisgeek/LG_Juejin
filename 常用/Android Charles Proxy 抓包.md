# Android Charles Proxy 抓包
- 抓包工具 Charles Proxy，是一款跨平台的 HTTP/HTTPS 代理服务器与网络封包分析工具，核心是通过拦截转发网络请求实现抓包、调试与模拟，广泛用于移动开发、Web 开发与接口联调
- 可以实现弱网模拟、Mock 数据、压力测试等场景
- 可以通过安装 Charles 根证书来解决 HTTPS 抓包问题（HTTPS 请求显示 unknown）

## 启动代理服务​
- Charles 启动后，会自动配置系统代理（默认端口：8888），此时电脑的所有网络请求访问都会经过 Charles 转发，可在 Charles 窗口查看请求记录（比如访问 https://api.example.com，由于 HTTPS 请求是加密的，默认无法查看内容，需要额外安装 Charles 根证书）
- 可以通过菜单 Proxy -> Windows proxy 取消勾选来关闭这个系统代理

## 配置手机代理
- 通过 Charles 菜单的 Help -> Local IP Addresses 功能可以查看电脑的 IP（选中后可以用 Ctrl + C 进行复制），或通过操作系统设置自行查看
- 通过 Charles 菜单的 Proxy -> Proxy Settings 设置监听端口（默认端口：8888）
- 手机上设置代理：设置 -> WLAN 设置 -> 连上指定的 Wifi（需要确保与电脑在同一局域网） -> 网络详情 -> 代理 -> 选择手动 -> 主机名填入电脑的 IP，端口填入 8888 后保存

## HTTPS 请求抓包
- 通过 Charles 菜单的 Proxy -> SSL Proxying Settings -> 勾选 Enable SSL Proxying 来开启 SSL 代理，选中对应标签页，在下方点击 + 号添加一条规则，Host 填写目标域名（比如输入通配符 *，代理所有 HTTPS 请求），Port 输入 443（HTTPS 默认端口），也可以根据需要添加特定域名（比如 *.example.com）
- 如果抓包电脑端的请求，通过菜单的 Help -> SSL Proxying -> Install Charles Root Certificate，按照向导来安装根证书
- 如果抓包手机端的请求，手机上通过浏览器访问 chls.pro/ssl 网址（Charles 提供的证书下载地址，可以通过 Charles 菜单的 Help -> SSL Proxying -> Install Charles Root Certificate on a Mobile Device or Remote Browser 查看信息），下载并安装 Charles  根证书，打开手机设置 -> 安全 -> 更多安全设置 -> 加密与凭据 -> 安装证书（从存储设备安装） -> CA 证书 -> 在本地 SD 存储中找到已下载的证书文件
- 也可以通过 Charles 菜单的 Help -> SSL Proxying -> Save Charles Root Certificate，将证书保存为 .cer 文件（记得切换文件类型），自行进行手动导入。最终 Charles 里如果能看到请求的明文内容，就说明所有必需的配置都设置成功了
- 安卓从 Android 7.0（API 24）开始，出于安全考虑引入了安全机制，系统默认不再信任用户安装的 CA 证书，这意味着像 Charles、Fiddler 等抓包工具的 HTTPS 证书，会被 App 拒绝，导致无法抓包，解决方案就是修改 App 的 AndroidManifest.xml 中配置的网络安全配置文件，指定在调试模式下信任用户证书
```xml
<application
    android:name=".MyApplication"
    android:networkSecurityConfig="@xml/network_security_config">
    <!-- 其他配置 -->
</application>
```

network_security_config.xml
```xml
<​?xml version="1.0" encoding="utf-8"?>
<network-security-config>
    <!-- debug 调试模式下信任证书（为了 Charles/Fiddler 抓包），此配置仅在 debug 版本生效（android:debuggable="true"），release 版本会自动失效，确保发布版本的安全性 -->
    <debug-overrides>
        <trust-anchors>
            <!-- 信任系统预装的证书 -->
            <certificates src="system" />
            <!-- 信任用户安装的证书 -->
            <certificates src="user" />
        </trust-anchors>
    </debug-overrides>
</network-security-config>
```

## 模拟弱网环境
- 用于测试应用在恶劣网络下的表现​，通过 Charles 菜单的 Proxy -> Throttling settings 勾选 Enable Throttling 启用限流（节流）功能，可精准模拟 3G、4G 等弱网环境，提供了常见的网络预设模板供快速选择

## 修改请求与响应
- Mock 数据，右键点击请求，选择 Breakpoints 设置断点，当请求发送时，Charles 会暂停，可修改请求头或参数后点执行继续，当响应到来时，Charles 会暂停，修改返回的 JSON 等数据后点执行继续

## 重复发送请求
- 右键点击请求，选择 Repeat 立即重复发送一次请求，选择 Repeat Advanced 可设置 Iterations 重复次数、Concurrency 并发量、Delay 延迟等参数后再进行执行，可以用于模拟压力测试
