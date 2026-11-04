# Flutter 架构

## 理论参考
### Flutter 官方架构指南
- UI 层、Data 层、Domain 可选、UseCase 按需
- app/app.dart + routing 挂载点
- features/data RepositoryImpl + DataSource + DTO
- features/domain Model + Repository 接口 + 按需 UseCase
- features/presentation View + ViewModel
- core/foundation Result/Exception 结构

### Riverpod 3.x 文档
- Notifier / AsyncNotifier 的用法（Notifier 即 ViewModel）
- bootstrap.dart 装配、main_* Flavor 入口、presentation ViewModel 实现

### Andrea Bizzotto
- Feature-first、Controller、可选 application 层、common 层 
- xfeatures/<feature> 按任务划分、application/ Service 按需建、shared/ 只放真共享防垃圾场

### 六边形架构
- core/ports + app/adapters 
- core/ 只放接口不放实现、app/ 放实现和装配收口

### Clean Architecture
- 依赖方向规则：app -> features -> shared -> core 单向依赖
- core 禁止依赖 Flutter