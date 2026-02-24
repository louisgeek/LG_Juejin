# Android ViewBinding

```java
public class MainActivity extends AppCompatActivity {
    //Activity 的 binding 只创建一次，Activity 销毁时自动回收
    private ActivityMainBinding binding;
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        binding = ActivityMainBinding.inflate(getLayoutInflater());
        //setContentView(R.layout.activity_main);
        setContentView(binding.getRoot());
    }
}
```
```java
public class MyFragment extends Fragment {
    //Fragment 的 binding 可能出现多次创建和销毁，onDestroyView 时 binding 必须置空
    private FragmentMyBinding binding;
    @Override
    public View onCreateView(@NonNull LayoutInflater inflater, 
                             ViewGroup container, 
                             Bundle savedInstanceState) {
        //return inflater.inflate(R.layout.fragment_blank, container, false);
        binding = FragmentMyBinding.inflate(inflater, container, false);
        return binding.getRoot();
    }
    
    @Override
    public void onViewCreated(@NonNull View view, Bundle savedInstanceState) {
        super.onViewCreated(view, savedInstanceState);
        //使用 binding
    }
    
    @Override
    public void onDestroyView() {
        super.onDestroyView();
        binding = null;  //防止内存泄漏
    }
}
```

## Kotlin
```kotlin
class MainActivity : AppCompatActivity() {
    private lateinit var binding: ActivityMainBinding
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)
    }
}
```
```kotlin
class MyFragment : Fragment() {
    private var _binding: FragmentMyBinding? = null
    private val binding get() = _binding!!
    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View {
        _binding = FragmentMyBinding.inflate(inflater, container, false)
        return binding.root
    }
    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        //使用 binding
    }
    override fun onDestroyView() {
        super.onDestroyView()
        _binding = null  //防止内存泄漏
    }
}
```

## 利用委托配合 ::inflate
```kotlin
//Activity 采用 lazy 委托
inline fun <T : ViewBinding> ComponentActivity.viewBinding(
    crossinline bindingInflater: (LayoutInflater) -> T
) = lazy(LazyThreadSafetyMode.NONE) { //延迟初始化（主线程访问，无需同步，用 NONE 提升性能）
    bindingInflater(layoutInflater) //其实就是 bindingInflater.invoke(layoutInflater)
}
//使用
class MainActivity : ComponentActivity() {
    //private val binding by lazy(LazyThreadSafetyMode.NONE) { ActivityMainBinding.inflate(layoutInflater) }
    //首次访问 binding 时自动调用 inflate
    private val binding by viewBinding(ActivityMainBinding::inflate)
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(binding.root)
    }
}
```

```kotlin
//Fragment 采用自定义属性委托，不用匿名类是为了调用 createView 方法
class FragmentViewBindingDelegate<T : ViewBinding>(
    private val bindingInflater: (LayoutInflater, ViewGroup?, Boolean) -> T,
    private val fragment: Fragment
) : ReadOnlyProperty<Fragment, T> {
    private var binding: T? = null
    //需要在在 onCreateView 方法中调用
    fun createView(inflater: LayoutInflater, container: ViewGroup?): View {
        binding = bindingInflater(inflater, container, false)
        fragment.viewLifecycleOwner.lifecycle.addObserver(object : DefaultLifecycleObserver {
            override fun onDestroy(owner: LifecycleOwner) {
                binding = null //自动置空，防止内存泄漏
            }
        })
        return binding!!.root
    }
    //属性委托的 getValue 方法，通过 by 访问 binding 时调用
    override fun getValue(thisRef: Fragment, property: KProperty<*>): T {
        return binding ?: throw IllegalStateException(
            "Binding only valid between onCreateView and onDestroyView"
        )
    }
}
fun <T : ViewBinding> Fragment.viewBindingDelegate(
    bindingInflater: (LayoutInflater, ViewGroup?, Boolean) -> T
) = FragmentViewBindingDelegate(bindingInflater, this)
//使用
class MyFragment : Fragment() {
    //创建委托实例
    private val bindingDelegate = viewBindingDelegate(FragmentMyBinding::inflate)
    //通过 by 委托访问
    private val binding by bindingDelegate
    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View = bindingDelegate.createView(inflater, container)
    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        //使用 binding
    }
}
```

## 利用委托配合 ::bind
- 不推荐
```kotlin
//Fragment 采用自定义属性委托
inline fun <T : ViewBinding> Fragment.viewBinding(
    crossinline bindingBinder: (View) -> T
) = object : ReadOnlyProperty<Fragment, T> {
    private var binding: T? = null
    override fun getValue(thisRef: Fragment, property: KProperty<*>): T {
        if (binding == null) {
            //从 Fragment 中获取已创建的 View
            val view = thisRef.view ?: throw IllegalStateException(
                "Binding only valid between onCreateView and onDestroyView"
            )
            //使用 bind 绑定已存在的 View
            binding = bindingBinder(view)
            // 监听生命周期，自动置空
            thisRef.viewLifecycleOwner.lifecycle.addObserver(object : DefaultLifecycleObserver {
                override fun onDestroy(owner: LifecycleOwner) {
                    binding = null // 自动置空，防止内存泄漏
                }
            })
        }
        return binding!!
    }
}

//使用
class MyFragment : Fragment() {
    //使用 bind 而不是 inflate（首次访问 binding 时，自动调用 FragmentMyBinding.bind(view) 方法）
    private val binding by viewBinding(FragmentMyBinding::bind)
    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View {
        //手动 inflate 布局
        return inflater.inflate(R.layout.fragment_my, container, false)
        //或者这样写也行
        //return FragmentMyBinding.inflate(inflater, container, false).root
    }
    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        //使用 binding
    }
}
```