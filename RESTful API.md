# RESTful API
- Representational State Transfer 表述性状态转移，RESTful 即表示符合 REST 特性的（如果一个架构符合 REST 原则，就称为 RESTful 架构）
- RESTful API 是一种基于 HTTP 标准的设计风格和架构原则（接口设计规范），其核心思想是利用 URI 资源定位（统一的接口）和标准 HTTP 方法实现客户端与服务器之间的数据通信交互（通过 URI 统一资源标识符，通过标准 HTTP 方法操作资源），具有跨平台、简洁轻量级、灵活可扩展和易维护的特点
- 统一接口：使用 GET、POST、PUT、DELETE 等标准 HTTP 方法表示操作，每个资源都有一个唯一的 URI 来标识
- 无状态性：每次请求都必须包含足够的必要信息来完成该请求，服务器不维护上下文信息，更加灵活可扩展性（服务器无需维护 Session 会话状态），提升安全性（服务器不存储身份验证凭证）

## 设计规范
- 使用名词而非动词（每个地址代表一种资源）：比如用 GET /users 表示获取（读取）用户列表而不是 GET /getUsers
- 版本控制：/api/v1/users 在路径中添加版本号
- 资源嵌套（嵌套层级不超过两层）：/users/123/orders 表示指定用户的订单列表
- GET 获取资源（Read）：使用 GET /orders 获取订单列表（名词复数形式），使用 GET /orders/123 获取指定订单
- POST 创建新资源（Create）：使用 POST /orders 创建一个新的订单
- PUT 完全更新资源（Update，覆盖整个资源）：使用 PUT /orders/123 修改指定订单
- PATCH 部分更新资源（Update）：使用 PATCH /orders/123 修改指定订单
- DELETE：删除资源（Delete）：使用 DELETE /orders/123 删除指定订单

## 总结
- RESTful 通过标准化接口和资源化设计，是目前非常主流的 API 设计风格，尤其适用于需要高扩展性和跨平台兼容的场景
- 通过标准 HTTP 方法定义操作类型，结合 URI 统一资源标识符来实现标准化数据交互
- RESTful 理念就是将系统中的一切视为资源，使用标准 HTTP 方法（比如 GET、POST、PUT、DELETE 等）对资源进行增删改查（CRUD）操作