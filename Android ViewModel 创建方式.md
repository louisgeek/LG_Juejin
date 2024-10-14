# Android ViewModel 创建方式

- 1 ViewModelProvider 常规方式
```kotlin
 private lateinit var mainViewModel: MainViewModel
 
 override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        //如果是 Activity 就写在 onCreate 方法里
        mainViewModel = ViewModelProvider(this).get(MainViewModel::class.java)
        //
        mainViewModel = ViewModelProvider(requireActivity()).get(MainViewModel::class.java)
}
```

- 2 by lazy 委托
```kotlin
 //顶层函数，默认返回的是一个 Lazy 接口实现的 SynchronizedLazyImpl 类，是懒汉式的同步实现 ，线程安全，内部值只会初始化一次
 private val mainViewModel by lazy {
     ViewModelProvider(this).get(MainViewModel::class.java)
 }
```

- 3 by viewModels 或 activityViewModels
```kotlin
 //implementation "androidx.fragment:fragment-ktx:1.6.2"
 //扩展函数，返回的是一个 Lazy 接口实现的 ViewModelLazy 类，而且里面对返回的 viewModel 也做了缓存 
 private val mainViewModel: MainViewModel by viewModels() //Fragment 里用
 private val mainViewModel: MainViewModel by activityViewModels() //Fragment 里用

 //implementation "androidx.activity:activity-ktx:1.7.2"
 private val mainViewModel: MainViewModel by viewModels() //Activity 里用
```