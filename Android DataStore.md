# Android DataStore
- DataStore 是一种现代化的数据存储解决方案，提供了支持异步、类型安全和可观察的数据持久化管理方式，旨在替代传统的 SharedPreferences
- 基于 Kotlin 协程和 Flow 实现异步读写数据操作（所有读写操作默认在后台线程执行，避免主线程阻塞）
- 支持存储键值对（Preferences DataStore）或复杂的数据结构（Proto DataStore，需要 Protocol Buffers 支持）
- DataStore 采用类型安全的异步 API

主要分为  和 
支持使用 Kotlin 协程和 Flow 进行数据操作，适用于存储键值对或复杂的数据结构

基于 Kotlin 协程 和 Flow 实现异步、一致的事务存储
基于 Kotlin 协程和 Flow 实现异步读写，解决了 SharedPreferences 的同步阻塞、类型不安全等问题


```groovy
//Preferences DataStore
implementation 'androidx.datastore:datastore-preferences:1.1.1'
// 若使用 Proto DataStore，则添加
implementation "androidx.datastore:datastore:1.0.0"

// Proto DataStore（）
implementation "androidx.datastore:datastore:1.0.0"
implementation "com.google.protobuf:protobuf-javalite:3.21.12"

```

## 写入数据
```kotlin
val NAME_KEY = preferencesKey<String>("name")
val AGE_KEY = intPreferencesKey("age")
//
suspend fun saveUserData(context: Context, name: String, age: Int) {
    //DataStore#edit 函数
    context.dataStore.edit { preferences ->
        preferences[NAME_KEY] = name
        preferences[AGE_KEY] = age
    }
}
```

## 读取数据
```kotlin
//返回 Flow
val nameFlow: Flow<String> = context.dataStore.data
    .map { preferences ->
        preferences[NAME_KEY] ?: "DefaultName"
    }
//
// 读取数据  
context.dataStore.data.first()[NAME_KEY]   
//
val userDataFlow: Flow<Prefs> = context.dataStore.data
userDataFlow.map { preferences ->
    val name = preferences[NAME_KEY] ?: "DefaultName"
    val age = preferences[AGE_KEY] ?: 0
    //处理数据
}

```