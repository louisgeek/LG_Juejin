



| 类型 | 统一命名 | Kotlin/Android | Flutter/Dart |
|---|---|---|---|
| **网络数据模型** | `XxxDto` | `XxxDto` | `XxxModel` |
| **本地DB模型** | `XxxEntity` | `XxxEntity`（Room） | `XxxModel` / `XxxCompanion`（drift）|
| **Domain模型** | `Xxx`（无后缀） | `Xxx` | `Xxx` / `XxxEntity` |
| **UI状态** | `XxxUiState` | `XxxUiState` | `XxxState` |
| **UI事件（一次性）** | `XxxEvent` | `XxxEvent` | `XxxEvent` |
| **ViewModel** | `XxxViewModel` | `XxxViewModel` | `XxxViewModel` / `XxxCubit` / `XxxBloc` |
| **UseCase** | `XxxUseCase` | `XxxUseCase` | `XxxUseCase` |
| **Repository接口** | `XxxRepository` | `XxxRepository` | `XxxRepository` |
| **Repository实现** | `XxxRepositoryImpl` | `XxxRepositoryImpl` | `XxxRepositoryImpl` |
| **远程数据源** | `XxxRemoteDataSource` | `XxxApiService` / `XxxRemoteDataSource` | `XxxRemoteDataSource` |
| **本地数据源** | `XxxLocalDataSource` | `XxxDao`（Room）/ `XxxLocalDataSource` | `XxxLocalDataSource` |
| **DI模块** | `XxxModule` | `XxxModule` | `XxxModule` |
| **页面** | `XxxScreen` | `XxxScreen` | `XxxPage` / `XxxScreen` |
| **网络模型 → Domain** | `toDomain()` | `toDomain()` | `toEntity()` |
| **本地DB模型 → Domain** | `toDomain()` | `toDomain()` | `toEntity()` |
| **Domain → 网络模型** | `toDto()` | `toDto()` | `toModel()` |
| **Domain → 本地DB模型** | `toEntity()` | `toEntity()` | `toModel()` |
| **Domain → UI状态** | `toUiState()` | `toUiState()` | `toState()` |
