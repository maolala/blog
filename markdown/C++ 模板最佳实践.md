## 模板参数
Template Parameters有三种类型：
>* Type parameters(类型参数)；这种参数最常用 `template <typename T>`
>* Non-type parameters(非类型参数 - 整数或枚举类型 编译期常数、pointer类型、reference类型)； `template <size_t N = 1>` `template <typename T, typename T::Allocator* Allocator>`
Non-type parameters总是右值，不能取被取址，也不能被赋值
>* Template template parameters(双重模板参数) `template <typename T, template <typename ELEM, typename ALLOC = std::alloc<ELEM>> class CONT = std::vector>`

Template引入的“双阶段名称查找（Two phase name lookup）”堪称是C++中最黑暗的角落，名称查找会在模板定义和实例化时各做一次，分别处理非依赖性名称和依赖性名称的查找。

实例化模板匹配的时候，`通过原型得到模板实参类型，然后计算偏特化和全特化形式的匹配，如果无法匹配则使用原型`

## Template template parameters

```c++
#include <iostream>
#include <vector>
#include <deque>
#include <typeinfo>
#include <cxxabi.h>

template <typename T,
        template<typename ELEM, typename ALLOC = std::allocator<ELEM>>
        class CONT = std::deque >
class NestTemplate{
public:
    NestTemplate() {
        std::cout << abi::__cxa_demangle(typeid(CONT<T>).name(), nullptr, nullptr, nullptr) << std::endl;
    }
    template <typename N>
    void nest_func(N n) {
        std::cout << n << std::endl;
    }
private:
    CONT<T> elem_;
};

int main() {
    NestTemplate<int> t;
//    t.nest_func(5);

    std::cout << abi::__cxa_demangle(typeid(int).name(), nullptr, nullptr, nullptr) << std::endl;

    return 0;
}
```

嵌套模板，模板类型作为模板参数
template template argumnets(实参)必须完全匹配其对应参数

## avoid array to pointer
模板参数可以防止`array转为pointer`的转型动作，常被称为退化
```c++
template <typename T>
void avoid_decay_func(T& args) { //这里必须是引用传递
    std::cout << typeid(T&).name() << std::endl;
    std::cout << abi::__cxa_demangle(typeid(T&).name(), nullptr, nullptr, nullptr) << std::endl;
    std::cout << sizeof(args) << std::endl;
}
int main() {
    char type_array[10]{'0'};
    avoid_decay_func(type_array);
}
/*
A10_c
char [10]
10
*/
```
char array 与 char pointer

## type has member type X

在编译器判定某给定类型T是否有member type x

```c++
template <typename T>
auto contains_of(typename T::X const*) -> char {
    return static_cast<char>(0);
}
template <typename T>
auto contains_of(...) -> char*{
    return static_cast<char*>(0);
}

#define type_has_member_type_X(T) \
    (sizeof(contains_of<T>(0)) == 1)

struct Test{
    typedef decltype(nullptr) X;
};

int main() {
    std::cout << type_has_member_type_X(Test) << std::endl; //1
    return 0;
}
```

此宏的功用是判别编译器实例化contains_of<T>(0)时选中第一个function template或第二个，编译器把算式值绑定至template parameters时
>* 最佳匹配：重载解析规则会优先考虑`把0转换为一个null指针`，而不考虑`把0值当做省略号(ellipsis)参数`，重载解析规则中最后才考虑是否匹配的参数形式
>* SFINAE原则：替换失败并非错误
        
## 模板与设计
>* Generic programming(泛型编程)：使用泛化参数进行编程，以便找出高效算法之最抽象表述
>* Traits(特征、类型提取)：traits来表达`与某给定主类型有所关联`的类型附加信息，相当于类型绑定特征
>* Policy classes(策略类别)
>* Meta-programming(超编程)
>* Expression templates(算式模板)
  
## 参考资料
《C++ Template 全览 - 侯捷 简体中文版》
