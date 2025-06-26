# C／C++

## C++ 中 :: 的作用
- 作用域解析运算符，作用域解析（类/命名空间）
- std::cout 可简化为 cout（假设已声明了 using namespace std;）
1 访问全局变量（::value）
2 访问命名空间中的成员（Math::add()）
3 访问类的静态成员（Counter::count）
4 定义类外的成员函数，在类外部通过类名::函数定义，指明所属（归属）类（Rectangle::area()）
5 基类成员调用（Base::func()）
