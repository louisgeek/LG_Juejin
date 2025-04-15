# Android Intent
- Intent 意图（请求）是一种用于组件间传递数据的对象，是组件间通信的桥梁，可以看作是一个消息传递对象，用于请求目标组件执行某个操作，通过描述操作类型、数据及和目标组件等实现（触发目标组件的执行）
- Intent 是一个轻量级的消息对象，可以用来启动 Activity、启动 Service 和发送广播等，Intent 传输数据的大小受 Binder 机制的制约限制（跨进程，通常小于 1 MB）

## ComponentName 组件名称
- 指定目标组件类名（显式 Intent），比如 DetailActivity.class

## Action 动作
- 描述操作类型，比如 Intent.ACTION_VIEW 查看数据和 Intent.ACTION_SEND 发送数据
- Action 可以自定义

## Data 数据
- 指定操作的数据，通常为 URI 格式（比如 Uri.parse("tel:123456") 拨打电话、Uri.parse("mailto:user@example.com") 发送邮件和 Uri.parse("https://www.example.com") 访问网址）

## Type
- 指定数据的 MIME 类型，比如 text/plain、image/jpeg

## Category 类别
- 用于指定组件应满足的额外条件，比如 Intent.CATEGORY_DEFAULT 默认行为、Intent.CATEGORY_LAUNCHER 启动入口

## Extras 附加数据
- 以键值对的形式传递额外数据，使用 intent.putExtra("key", "value") 添加数据

## Flags 标记
- 控制 Intent 的行为，比如 FLAG_ACTIVITY_NEW_TASK 新任务启动


## 显式 Intent
- 直接指定目标组件的类名
```java
//启动 Activity
Intent intent = new Intent(MainActivity.this, DetailActivity.class);
context.startActivity(intent);
```

```java
//启动 Service
Intent intent = new Intent(MainActivity.this, MyService.class);
context.startService(intent);
//停止 Service
context.stopService(intent);
```

## 隐式 Intent
- 通过 Action、Data 和 Category 等属性来描述需求，由系统自动匹配符合条件的组件，处理请求，提升代码复用性和灵活性
```java
//打开网址
Intent intent = new Intent();
intent.setAction(Intent.ACTION_VIEW);
intent.setData(Uri.parse("http://www.example.com"));
context.startActivity(intent);
```

```java
//发送文本
Intent shareIntent = new Intent(Intent.ACTION_SEND);
shareIntent.setType("text/plain");
shareIntent.putExtra(Intent.EXTRA_TEXT, "分享内容");
context.startActivity(Intent.createChooser(shareIntent, "选择分享方式"));
```

## Intent 检查
- 检查目标组件是否存在（是否有效），避免应用崩溃
```java
if (intent.resolveActivity(getPackageManager()) != null) {
    context.startActivity(intent);
} else {
    Toast.makeText(this, "无法执行此操作", Toast.LENGTH_SHORT).show();
}
```

## IntentFilter
- 在 AndroidManifest.xml 中声明组件支持的 Intent 类型，用于匹配隐式 Intent

```xml
<activity android:name=".DetailActivity">
    <intent-filter>
        <action android:name="android.intent.action.VIEW" />
        <category android:name="android.intent.category.DEFAULT" />
        <data android:scheme="http" />
    </intent-filter>
</activity>
```

## PendingIntent
- 封装 Intent 并延迟执行（比如通知、闹钟）
```java
Intent intent = new Intent();
intent.putExtra("key", "value");
PendingIntent pendingIntent = PendingIntent.getActivity(
    context, 0, intent, PendingIntent.FLAG_IMMUTABLE
);
```

## Bundle
- Bundle 是一个键值对集合，可以存储多种类型的数据，通常与 Intent 一起使用，用于传递多个参数或者复杂的数据结构
- Intent#putExtra 内部也是使用的 Bundle 来传输的数据
- Bundle 内部是由 ArrayMap 实现的
```java
Intent intent = new Intent(MainActivity.this, DetailActivity.class);
Bundle bundle = new Bundle();
bundle.putString("name", "Louis");
bundle.putInt("age", 25);
intent.putExtras(bundle);
context.startActivity(intent);
```