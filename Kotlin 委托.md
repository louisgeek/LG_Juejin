# Kotlin 委托

## 委托和代理
- Delegate 委托和 Proxy 代理有点大同小异：代理比较严格，一般是同一个对象（实现了同一个接口或继承了同一个类），委托一般是不同的对象，但通常也对外提供被委托对象的同名方法，委托引用了被委托对象
- 应用：Android 中 AppCompatActivity 委托给一个叫 AppCompatDelegate 的抽象类，AppCompatDelegateImpl 是其实现，其中 AppCompatActivity#setContentView 方法就是委托了 AppCompatDelegate#setContentView 方法，可以在其前后增加额外的处理逻辑

```java
public class AppCompatActivity extends FragmentActivity implements AppCompatCallback,
        TaskStackBuilder.SupportParentable, ActionBarDrawerToggle.DelegateProvider {

    private AppCompatDelegate mDelegate;

    @NonNull
    public AppCompatDelegate getDelegate() {
        if (mDelegate == null) {
            mDelegate = AppCompatDelegate.create(this, this);
        }
        return mDelegate;
    }

    @Override
    public void setContentView(@LayoutRes int layoutResID) {
        initViewTreeOwners();
        //
        getDelegate().setContentView(layoutResID);
    }      
}

```

- PS：从我个人理解来说，Kotlin 委托更像是上面描述的代理，或者我这么理解：代理要求比委托高，但也不妨碍我们让委托变得严苛起来，让委托和代理一样严苛的话，那么此时的委托是不是就算是和代理是一样的了？所以换句话说委托范畴可以更大，是不是可以包含代理？或者代理是不是可以看成是一种更特殊的委托？ —— 部分 kotlin 书籍里也是将委托直接当成代理的，另外一些 java 知识文章里，把被代理类（目标类）称为委托类

## by 关键字

by 是 Kotlin 中用来实现委托的，编译器会生成委托模式的模板代码，使我们能够更方便的利用聚合来替代继承
- 类委托（需要实现同一个接口，不能是抽象类）
- 属性委托（getValue/setValue 方法的委托）

## 类委托
- 将接口实现委托给另一个类对象
```kotlin
    interface IPlayer{
        fun start()
        fun stop()
    }
    class DoPlayer: IPlayer{
        override fun start() {
            Log.e(TAG, "start: do" )
        }

        override fun stop() {
            Log.e(TAG, "stop: do" )
        }

    }
    //by 子句表示将 player 保存在 DelegatePlayer 的对象实例内部
    class DelegatePlayer(private val player: IPlayer): IPlayer by player

 ```

 ```kotlin
      val player = DelegatePlayer(DoPlayer())
        player.start()
        player.stop()
```

```kotlin
//PS：直接写 DoPlayer()
 class DelegatePlayer2 : IPlayer by DoPlayer(){
        
 }
```

## 属性委托
- 将属性的 get/set 逻辑委托给另一个类对象
- 此类需要提供 getValue 或者 setValue 约定函数
```kotlin
   private var test03: String by Delegate()

    class Delegate {
        private var testStr: String? = null
        //operator重载运算符
        operator fun getValue(thisRef: Any, property: KProperty<*>): String {
            return testStr?:"tess"
        }

        operator fun setValue(thisRef: Any, property: KProperty<*>, s: String) {
            testStr = s
        }
    }

```

PS：写着麻烦？写完 by Delegate() 后，AS 会提示报错需要生成；或者实现自带的接口即可
```kotlin
    //val 属性实现 ReadOnlyProperty
    class Delegate2 :ReadWriteProperty<Any,String>{
        private var testStr: String? = null
        override fun getValue(thisRef: Any, property: KProperty<*>): String {
            return testStr?:"tess"
        }

        override fun setValue(thisRef: Any, property: KProperty<*>, value: String) {
            testStr = value
        }
    }

```

# 常见应用

```kotlin
    //简化项目里 ShareDataManage#save 和 ShareDataManage#get
    private var spBooleanValue by SpBoolean(SP_NAME, "key1")
    private var spStringValue by SpString(SP_NAME, "key2")
 
    fun getBooleanValue(): Boolean = spBooleanValue
    fun getStringValue(): String = spStringValue
 
    fun setSpValue(value: Boolean) {
        spBooleanValue = value
    }
 
    fun setSpValue(value: String) {
        spStringValue = value
    }


class SpString(private val spName: String, val key: String, private val defValue: String = "") {
    operator fun getValue(thisRef: Any?, property: KProperty<*>): String {
        val sp = context.getSharedPreferences(spName, Context.MODE_PRIVATE)
        return sp.getString(key, defValue)!!
    }
 
    operator fun setValue(thisRef: Any?, property: KProperty<*>, value: String) {
        val sp = context.getSharedPreferences(spName, Context.MODE_PRIVATE)
        sp.edit().putString(key, value).apply()
    }
}
```

## Kotlin 标准库里内置的
- lazy 顶层函数
- Delegates.notNull
- Delegates.observable //其实 ObservableProperty 就是在 setValue 赋值的前后插入抽象方法，而 Delegates.observable 就采用理其中的 afterChange
- Delegates.vetoable //其实 ObservableProperty 就是在 setValue 赋值的前后插入抽象方法，而 Delegates.vetoable 就采用理其中的 beforeChange，但是比较特殊的就是 beforeChange 返回一个布尔值，决定了后续是否继续执行赋值，默认返回 true，如果返回 false 值则不继续赋值

## by lazy
- 延迟委托
```kotlin

//lazy 是一个顶层函数，默认返回的是一个 Lazy 接口的实现类 SynchronizedLazyImpl，逻辑基本和 Java 的双重检验单例一致 ，线程安全，内部值只初始化一次
val gzLayout: GZLayout by lazy { 
    this.findViewById(R.id.gzLayout) 
}

```

但是咋一看  SynchronizedLazyImpl 类没有找到 getValue 或者 setValue 约定函数， 如何实现属性委托呢？其实 Lazy.kt 文件里自带一个 Lazy 接口的 getValue 扩展函数，所以可以这么实现

```kotlin
/**
 * An extension to delegate a read-only property of type [T] to an instance of [Lazy].
 *
 * This extension allows to use instances of Lazy for property delegation:
 * `val property: String by lazy { initializer }`
 */
@kotlin.internal.InlineOnly
public inline operator fun <T> Lazy<T>.getValue(thisRef: Any?, property: KProperty<*>): T = value
```


```kotlin
//扩展函数
fun <T : View> Activity.findView(id: Int) = lazy {
    this.findViewById<T>(id)
}
//
val gzLayout: GZLayout by findView(R.id.gzLayout)
```


## lateinit 关键字和 by Delegates.notNull
- 两者都需要自行保证在访问之前必须已经被初始化完成了
- lateinit 关键字只适用于引用类型
- by Delegates.notNull 适用于基本类型和引用类型

```kotlin
private lateinit var basicType01: Int //报错，不能用来修饰基本数据类型
private var basicType02: Int by Delegates.notNull() //可以这么写，不过访问之前必须自行保证已经被初始化
```