# 第一章 类型推导

## 1. 理解模板类型推导

- 对于模版类型推导，有引用的实参会被视为无引用，他们的引用会被忽略

  ```C++
  template<typename T>
  void f(ParamType param);
  int x=27;                       //x是int
  const int cx=x;                 //cx是const int
  const int& rx=x;                //rx是指向作为const int的x的引用
  f(x);                           //T是int，param的类型是int&
  f(cx);                          //T是const int，param的类型是const int&
  f(rx);                          //T是const int，param的类型是const int&
  ```

- 对于通用引用的推导，左值实参会被特殊对待，推导为左值引用

  ```C++
  template<typename T>
  void f(T&& param);              //param现在是一个通用引用类型
          
  int x=27;                       //如之前一样
  const int cx=x;                 //如之前一样
  const int & rx=cx;              //如之前一样
  
  f(x);                           //x是左值，所以T是int&，
                                  //param类型也是int&
  
  f(cx);                          //cx是左值，所以T是const int&，
                                  //param类型也是const int&
  
  f(rx);                          //rx是左值，所以T是const int&，
                                  //param类型也是const int&
  
  f(27);                          //27是右值，所以T是int，
                                  //param类型就是int&&
  ```

- 对于传值类型推导

  ```C++
  template<typename T>
  void f(T param);                //以传值的方式处理param
  int x=27;                       //如之前一样
  const int cx=x;                 //如之前一样
  const int & rx=cx;              //如之前一样
  
  f(x);                           //T和param的类型都是int
  f(cx);                          //T和param的类型都是int
  f(rx);                          //T和param的类型都是int
  ```

  const或volatile实参会被认为是non-const和non-volatile

- 在模板类型推导时，数组名或者函数名实参会退化为指针，除非它们被用于初始化引用

  ```C++
  void someFunc(int, double);         //someFunc是一个函数，
                                      //类型是void(int, double)
  
  template<typename T>
  void f1(T param);                   //传值给f1
  
  template<typename T>
  void f2(T & param);                 //传引用给f2
  
  f1(someFunc);                       //param被推导为指向函数的指针，
                                      //类型是void(*)(int, double)
  f2(someFunc);                       //param被推导为指向函数的引用，
                                      //类型是void(&)(int, double)
  ```


## 2. 理解auto类型推导

- auto 类型推导通常和模板类型推导相同，但是auto类型推导假定花括号初始化代表 std::list，而模版类型推导不允许这么做
- C++14 中auto允许出现在函数返回值或者lambda函数形参中，但是它的工作机制是模板类型推导那一套方案，而不是auto类型推导

## 3. 理解decltype

- decltype 总是不加修改的产生变量或者表达式的类型
- 对于T类型的不是单纯的变量名的左值表达式，decltype 总是产出T的引用即 T&
- C++14支持 decltype(auto)，就像 auto 一样，推导出类型，但是它使用decltype的规则进行推导

## 4. 学会查看类型推导结果

- 类型推断可以从IDE看出，从编译器报错看出，从Boost TypeIndex库的使用看出
- 这些工具可能既不准确也无帮助，所以理解C++类型推导规则才是最重要的

# 第二章 auto

## 5. 优先考虑auto而非显式类型声明

-   auto变量必须初始化，通常它可以避免一些移植性和效率性的问题，也使得重构更加方便
  - Size_type（64位） 与 unsigned（32位）
  - `std::unordered_map<std::string, int> ` 中的 `std::pair` 类型实际上是 `std::pair<const std::string, int>`
- auto类型的变量可能会踩陷阱，例如item2和item6

## 6. auto推导若非己愿，使用显式类型初始化惯用法

- 不可见的代理类可能会使 auto 从表达式中推导出“错误的”类型
  - `std::vector<bool>` 的 `operator[]` 返回一个行为类似于 `bool&` 的对象，但是C++禁止对 `bits` 的引用
- 显示类型初始器惯用法强制 auto 推导出你想要的结果 `static_cast<T>`

# 第三章 移步现代C++

## 7. 区别使用()和{}创建对象

```C++
Widget w1;              //调用默认构造函数

Widget w2 = w1;         //不是赋值运算，调用拷贝构造函数

w1 = w2;                //是赋值运算，调用拷贝赋值运算符（copy operator=）

```

- 花括号初始化是最广泛使用的初始化语法，它防止变窄转换，并且对于C++最令人头疼的解析有天生的免疫性
- 在构造函数重载决议中，编译器会尽最大努力将括号初始化与 `std::initializer_list` 参数匹配，即便其他构造函数看起来是更好的选择
- 对于数值类型的 `std::vector` 来说使用花括号初始化和圆括号初始化会造成巨大的不同
- 在模板类型选择使用圆括号初始化或使用花括号初始化创建对象是一个挑战

## 8. 优先考虑nullptr而非0和NULL

- 避免重载指针和整型

## 9. 优先考虑别名声明而非typedefs

- typedef不支持模板化，但是别名声明支持
- 病名模板避免了使用 `::type` 后缀，而且在模板中使用 `typedef` 还需要在前面加上  `typename`
- C++14 提供了 C++11 所有 type traits转换的别名声明版本

# 第四章 智能指针

## 18. 对于独占资源使用 std::unique_ptr

- 轻量级、快速的、只可移动（move only）的管理专有所有权语义资源的智能指针
- 默认情况，资源销毁通过 delete 实现，但是支持自定义删除器。有状态的删除器和函数指针会增加 `std::unique_ptr` 对象的大小
- 将 `std::unique_ptr` 转化为 `std::shared_ptr` 非常简单

## 19. 对于共享资源使用 std::shared_ptr

- 引用计数暗示性能问题：
  - 大小是原始指针的两倍，因为内部包含一个只想资源的原始指针，还包含一个指向资源的引用计数值的原始指针（其实是指向了 Control Block）
  - 引用计数的内存必须动态分配
  - 计数必须是原子性操作
- `std::unique_ptr` 把删除器视作类型的一部分，但共享指针不是，因为共享指针的删除器在 Control Block 中
- Control Block 的创建规则：
  - `std::make_shared` 总是创建一个控制块
  - 从独占指针构造出共享指针
  - 从原始指针上构造出共享指针
- 应当避免从原始指针变量上创建 `std::shared_ptr`

## 20. 当 std::shared_ptr 可能悬空时使用 std::weak_ptr

- 用 `std::weak_ptr` 替代可能会悬空的 `std::shared_ptr`
- `std::weak_ptr` 的潜在使用场景包括：缓存、观察者列表、打破 `std::shared_ptr` 环状结构

## 21. 优先考虑 std::make_unique 和 std::make_shared， 而非直接使用new

