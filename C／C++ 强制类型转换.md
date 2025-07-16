# C／C++ 强制类型转换
- 强制类型转换是一种显式地将一个类型的值转换为另一个类型的操作
- 分为 C 风格强制类型转换和 C++ 风格强制类型转换两种实现方式
- C 风格的类型转换比较简单，但缺乏类型检查，可能会引发安全问题，而 C++ 也继承兼容了 C 风格的类型转换方式，但不推荐使用
- C++ 为提升类型转换的安全性，引入了 4 种类型转换操作符，分别针对不同场景，提升代码安全性和可读性，使转换操作更加规范，明确意图，减少歧义

## (type)expression
- C 风格强制类型转换语法比较简单，但缺乏类型安全检查，可能会带来安全隐患（比如导致未定义的行为），同时也可能导致数据丢失
```c
//把 int 类型转换为 float 类型
int a = 10;
float b = (float)a;

//把 double 类型转换为 int 类型
double d = 3.14;
int i = (int)d; //截断为 3

//把 int* 类型转换为 void* 类型
int a = 10;
int* p = &a;
void* q = (void*)p;

//把 int* 类型转换为 char* 类型
int arr[5] = {1, 2, 3, 4, 5};
int* p1 = arr;
char* p2 = (char*)p1; 
```

## static_cast<type>(expression)
- 适用于基本类型之间的转换（比如 int 转 float）
- 适用于具有继承关系的类之间的指针/引用的转换
- 编译时类型检查，不进行运行时类型检查（错误的下行转换可能会导致未定义的行为）
```cpp
//把 int 类型安全地转换为 float 类型
int a = 10;
float b = static_cast<float>(a);  

//把 double 类型安全地转换为 int 类型
double d = 3.14;
int i = static_cast<int>(d);  //安全截断为 3

//父类 Base 和子类 Derived
Derived derived;
Base* basePtr = &derived; //上行转换（子类指针转换为父类指针），不需要强制转换
Derived* derivedPtr = static_cast<Derived*>(basePtr); //下行转换（父类指针转换为子类指针，需要确保类型正确）

//将一个引用类型转换为另一个引用类型
Derived derivedObj;
Base& baseRef = derivedObj;
Derived& derivedRef = static_cast<Derived&>(baseRef); //将父类引用 baseRef 转换为子类引用 derivedRef
```

## dynamic_cast<type>(expression)
- 适用于多态类（父类必须有虚函数）之间的转换，尤其是下行转换（父类转子类）的情况
- 运行时类型检查，如果转换失败，返回 nullptr（指针）或抛出 std::bad_cast 异常（引用）
```cpp
//父类 Base 和子类 Derived，运行时验证类型
Base* basePtr = new Derived();
Derived* derivedPtr = dynamic_cast<Derived*>(basePtr);  //转换成功，返回指针

Base* b = new Base();
Derived* d = dynamic_cast<Derived*>(b); //转换失败，d = nullptr
```

## const_cast<type>(expression)
- 适用于添加或删除对象的 const 或 volatile 限定符
- 如果修改 const 对象的值会导致未定义的行为
```cpp
const int a = 10;
int* p = const_cast<int*>(&a); //移除 const 属性，使得可以修改 a 的值
*p = 20; //未定义的行为（a 本身不可修改）
```

## reinterpret_cast<type>(expression)
- 是一种底层（低层次、低级别）的类型转换（比如整数与指针间的转换、指针类型之间的转换），低级位模式重新解释，直接操作内存，极易引发内存解释错误，需谨慎使用
- 可能导致未定义的行为、除非绝对必要（比如硬件编程），否则应避免使用
```cpp
//reinterpret 重新解释、重新诠释
int a = 10;
int* p = &a;
char* b = reinterpret_cast<char*>(p); //将 int 指针重新解释为 char 指针
```

## 总结
- 优先使用 C++ 风格新的强制类型转换，尽量避免采用 C 风格的
- 优先使用 static_cast，增强类型安全
- 多态类之间的转换必须用 dynamic_cast，确保类型安全，防止运行时错误
- 仅在必要时使用 reinterpret_cast，并严格验证其正确性