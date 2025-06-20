# Java Creational 创建型模式之 Builder 建造者模式
- 建造者模式是一种创建型设计模式，其核心思想是将一个复杂对象的构建过程与其表示分离，允许你分步骤构建复杂的对象，使得同样的构建过程可以创建出不同的表示形式，特别适合创建具有多个可选参数（属性）的对象，简化了对象的创建过程，提高代码的可读性和可维护性
- 链式调用：利用链式调用使代码更简洁、易读、灵活，方便配置可选参数（逐步设置属性）
- 封装性：将对象复杂的构建过程与其表示分离，隐藏了创建对象的具体细节，保持产品类的简洁
- 不可变性：构造函数私有化，只能通过 Builder 构建对象

产品类
```java
class Dialog {
    private String title;
    private String message;

    public String getTitle() {
        return title;
    }

    public String getMessage() {
        return message;
    }

    //私有化构造函数，只能通过 Builder 创建
    private Dialog(Builder builder) {
        this.title = builder.title;
        this.message = builder.message;
    }

    //静态内部建造者类
    public static class Builder {
        private Context context;
        private String title;
        private String message;

        public Builder(Context context) {
            //必要参数通过构造函数传入
            this.context = context;
        }

        public Builder setTitle(String title) {
            this.title = title;
            return this;
        }

        public Builder setTitle(int titleResId) {
            this.title = context.getString(titleResId);
            return this;
        }

        public Builder setMessage(String message) {
            this.message = message;
            return this;
        }

        public Builder setMessage(int messageResId) {
            this.message = context.getString(messageResId);
            return this;
        }

        //最终构建
        public Dialog build() {
            return new Dialog(this);
        }
    }

}
```

使用
```java
public static void main(String[] args) {
    //使用建造者构建
    Dialog dialogA = new Dialog.Builder(null)
            .setTitle("title")
            .setMessage("message")
            .build();

    Dialog dialogB = new Dialog.Builder(null)
            .setTitle("title")
            .build();
}
```

## 特点
- 分离构建逻辑，降低耦合度
- 通过链式调用替代多参数构造函数重载
- 通过不同配置的组合，生成对象不同表示
- 需额外定义 Builder 类，增加维护成本，简单对象使用可能会复杂化代码
