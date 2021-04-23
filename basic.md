### 类型推导

```
//函数模板
template<typename T>
void f(ParamType  param);

//调用
f(expr)
```

#### 1. ParamType是一个指针或引用，但不是万能引用.
* 若expr具有引用型别，先将引用部分忽略
* 然后对expr的型别和ParamType的型别进行模式匹配，来决定T的型别

```
template<typename T>
void f(T& param);

int x = 27;                     //f(x)   T: int,       param: int 
const int cx = x;               //f(cx)  T：const int, param: const int&
const int& rx = x;              //f(rx)  T: const int, param: const int&


template<typename T>
void f(const T& param);

int x = 27;                     //f(x)   T: int, param: const int& 
const int cx = x;               //f(cx)  T：int, param: const int&
const int& rx = x;              //f(rx)  T: int, param: const int&
```

#### 2. ParamType是一个万能引用.
* 若expr是个左值，T和ParamType都会被推导为左值引用。(T被推导为引用类型的唯一情形;尽管声明时使用的时右值引用语法，它的型别推导却是左引用值)
* 若expr是个右值，按1中的规则。

```
template<typename T>
void f(T&& param);

int x = 27;                     //f(x)   T: int&,       param: int& 
const int cx = x;               //f(cx)  T：const int&, param: const int&
const int& rx = x;              //f(rx)  T: const int&, param: const int&
                                //f(27)  T: int,        param: int&&
```

#### 3. ParamType既非指针也非引用
* 若expr具有引用型别，先将引用部分忽略
* 若expr是个const/volatile对象，也忽略之。

```
template<typename T>
void f(T param);

int x = 27;                     //f(x)   T: int,         param: int 
const int cx = x;               //f(cx)  T：int,         param: int
const int& rx = x;              //f(rx)  T: int,         param: int
const char* cosnt ptr = "test"; //f(ptr) T: const char*, param: const char*
```

#### 数组实参，函数实参

* 多数语境下，数组会退化成指涉到其首元素的指针。

```
const char name[] = "test";
const char *ptr = name; //退化为指针

template<typename T>
void f(T param)

f(name)  // T: const char *

template<typename T>
void f(T& param)

f(name)  // T: const char[5], param: const char(&)[5] 
```

获取数组大小:
```
template<typename T,std::size_t N>
constexpr std::size_t array_size(T (&)[N]) noexcept
{
    return N;
}
```

* 函数型别也会退化为指针。
```
void func(int ,double);

template<typename T>
void f(T param)         //f(func) param:void (*)(int,double)

template<typename T>
void f(T& param)        //f(func) param:void (&)(int,double) 
```

### auto
* 与上面推导类似

```
auto x = 27;                    // int
const auto cx = x;              // const int
const auto& rx = x;             // const int&

auto&& y1 = x;                  // int&
auto&& y2 = cx;                 // const int&
auto&& y3 = 27;                 // int&&

//唯一不同之处：统一初始化，std::initializer_list<T>，发生两次推导
auto x2 = {27};  //std::initializer_list<int> ,值为{27}
auto x3{27};     //同上
```

c++14允许使用auto来说明函数返回值需要推导，以及lambda表达式也会在形参声明中用到。但却使用模板类型推导。
```
auto create()
{
    return {1,2,3};   //error
}


std::vector<int> v;
auto r =[&v](const auto& nv){v = nv;};

r({1,2,3});     //error
```

### decltype
* 声明那些返回值型别依赖于形参型别的函数模板。 

```
//c++14 
template<typename Container,typename Index>
decltype(auto) authAndAccess(Container,typename Index)
{
    authenticateUser();
    return std::forward<Container>(c)[i];
}

//c++11
template<typename Container,typename Index>
decltype(auto) authAndAccess(Container,typename Index)->decltype(std::forward<Container>(c)[i])
{
    authenticateUser();
    return std::forward<Container>(c)[i];
}
```

例外：
```
decltype(auto) f()
{
    int x = 0;
    return (x);         //decltype(auto)是int&
}
```

###  C++11特种成员函数的机制

1. 默认构造函数：仅当类中不包含用户声明的构造 函数时
2. 析构函数：noexcept，仅当基类的析构函数为虚的，派生类的析构函数才是虚的
3. 复制构造函数：
   * 运行期：按成员进行非静态数据成员的复制构造。仅当类中不包含用户声明的复制构造函数时才生成。
   * 如果该类声明了移动操作，则复制构造函数将被删除。
4. 复制赋值运算符：
   * 运行期：按成员进行非静态数据成员的复制构造。仅当类中不包含用户声明的复制赋值运算符时才生成。
   * 如果该类声明了移动操作，则复制赋值运算符将被删除。
5. 移动构造函数和移动赋值运算符
   * 按成员进行非静态数据成员的移动操作 
   * 该类未声明任何复制操作，任何移动操作，任何析构操作才生成
6. 成员函数模板在任何情况下都不会抑制特种成员函数的生成

### std::move std::forward
1. move
* 告诉编译器该对象具备可移动的条件，简化了对象是否可移动的表述
* 把实参强制转换为右值
* 如果想取得对某个对象执行移动操作的能力，则不能将其声明为 常量，因为针对常量对象执行的移动操作将会变换成复制操作。
```
template<typename T>
decltype(auto) move(T&& param)
{
    using return_type = remove_reference_t<T>&&;
    return static_cast<return_type>(param);
}
```
2. forward
* 仅当传入的实参被绑定到右值时，才针对该实参实施向右值类型的强制类型转换
```
template<typename T>
T&& forward(remove_reference_t<T>& param)
{
    return static_cast<T&&>(param);
}
```

* 引用折叠语境：模板实例化，auto类型生成，创建和运用typedef和别名声明，decltype。
  
