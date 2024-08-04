---
title: C++
date: 2023-07-24
tags: C++
categories: 编程语言
---

# C++中map与unordered_map的区别

## 头文件

- map: `#include <map>`
- unordered_map: `#include <unordered_map>`

## 内部实现机理

- map： map内部实现了一个红黑树，该结构具有自动排序的功能，因此map内部的所有元素都是有序的，红黑树的每一个节点都代表着map的一个元素，因此，对于map进行的查找，删除，添加等一系列的操作都相当于是对红黑树进行这样的操作，故红黑树的效率决定了map的效率。
- unordered_map: `unordered_map`内部实现了一个哈希表，因此其元素的排列顺序是杂乱的，无序的

unordered_map和map类似，都是存储的key-value的值，可以通过key快速索引到value。不同的是unordered_map不会根据key的大小进行排序，存储时是根据key的hash值判断元素是否相同，即unordered_map内部元素是无序的，而map中的元素是按照二叉搜索树存储，进行中序遍历会得到有序遍历。

所以使用时map的key需要定义`operator<`。而unordered_map需要定义`hash_value`函数并且重载`operator==`。但是很多系统内置的数据类型都自带这些，那么如果是自定义类型，那么就需要自己重载`operator<`或者`hash_value()`了。

## 优缺点以及适用处

|        | map                                                          | unordered_map                                                |
| :----- | :----------------------------------------------------------- | :----------------------------------------------------------- |
| 优点   | 有序性，这是map结构最大的优点，其元素的有序性在很多应用中都会简化很多的操作红黑树，内部实现一个红黑书使得map的很多操作在lgnlgn的时间复杂度下就可以实现，因此效率非常的高 | 因为内部实现了哈希表，因此其查找速度非常的快                 |
| 缺点   | 空间占用率高，因为map内部实现了红黑树，虽然提高了运行效率，但是因为每一个节点都需要额外保存父节点，孩子节点以及红/黑性质，使得每一个节点都占用大量的空间 | 哈希表的建立比较耗费时间                                     |
| 适用处 | 对于那些有顺序要求的问题，用map会更高效一些                  | 对于查找问题，unordered_map会更加高效一些，因此遇到查找问题，常会考虑一下用unordered_map |

## 结论

如果需要内部元素自动排序，使用`map`，不需要排序使用`unordered_map`

## note:

对于`unordered_map`或者`unordered_set`容器，其遍历顺序与创建该容器时输入元素的顺序是不一定一致的，遍历是按照哈希表从前往后依次遍历的

## 参考

1. [c++中map与unordered_map的区别](https://blog.csdn.net/batuwuhanpei/article/details/50727227)
2. [C++11 新特性： unordered_map 与 map 的对比](https://www.cnblogs.com/NeilZhang/p/5724996.html)



# C++ 11之Lambda表达式

C++11的一大亮点就是引入了Lambda表达式。利用Lambda表达式，可以方便的定义和创建匿名函数,用以替换独立函数或者函数对象，并且使代码更可读。但是从本质上来讲，lambda表达式只是一种语法糖，因为所有其能完成的工作都可以用其它稍微复杂的代码来实现。但是它简便的语法却给C++带来了深远的影响。如果从广义上说，lamdba表达式产生的是函数对象。在类中，可以重载函数调用运算符()，此时类的对象可以将具有类似函数的行为，我们称这些对象为函数对象（Function Object）或者仿函数（Functor）。对于C++这门语言来说来说，“Lambda表达式”或“匿名函数”这些概念听起来好像很深奥，但很多高级语言在很早以前就已经提供了Lambda表达式的功能，如C#，Python等。今天，我们就来简单介绍一下C++中Lambda表达式的简单使用。

## 声明Lambda表达式

Lambda表达式完整的声明格式如下：

```
[capture list] (params list) mutable exception-> return type { function body } 
```

各项具体含义如下

- capture list：捕获外部变量列表
- params list：形参列表
- mutable指示符：用来说用是否可以修改捕获的变量
- exception：异常设定
- return type：返回类型
- function body：函数体

此外，我们还可以省略其中的某些成分来声明“不完整”的Lambda表达式，常见的有以下几种：

| 序号 | 格式                                                        |
| :--- | :---------------------------------------------------------- |
| 1    | [capture list] (params list) -> return type {function body} |
| 2    | [capture list] (params list) {function body}                |
| 3    | [capture list] {function body}                              |

其中：

- 格式1声明了const类型的表达式，这种类型的表达式不能修改捕获列表中的值。
- 格式2省略了返回值类型，但编译器可以根据以下规则推断出Lambda表达式的返回类型： （1）：如果function body中存在return语句，则该Lambda表达式的返回类型由return语句的返回类型确定； （2）：如果function body中没有return语句，则返回值为void类型。
- 格式3中省略了参数列表，类似普通函数中的无参函数。

Lambda表达式的一个实例

```
#include <iostream>
#include <vector>
#include <algorithm>
using namespace std;

bool cmp(int a, int b)
{
    return  a < b;
}

int main()
{
    vector<int> myvec{ 3, 2, 5, 7, 3, 2 };
    vector<int> lbvec(myvec);

    sort(myvec.begin(), myvec.end(), cmp); // 旧式做法
    cout << "predicate function:" << endl;
    for (int it : myvec)
        cout << it << ' ';
    cout << endl;

    sort(lbvec.begin(), lbvec.end(), [](int a, int b) -> bool { return a < b; });   // Lambda表达式
    cout << "lambda expression:" << endl;
    for (int it : lbvec)
        cout << it << ' ';
}
```

在C++11之前，我们使用STL的sort函数，需要提供一个谓词函数。如果使用C++11的Lambda表达式，我们只需要传入一个匿名函数即可，方便简洁，而且代码的可读性也比旧式的做法好多了。

下面，我们就重点介绍一下Lambda表达式各项的具体用法。

## 捕获外部变量

Lambda表达式可以使用其可见范围内的外部变量，但必须明确声明（明确声明哪些外部变量可以被该Lambda表达式使用）。那么，在哪里指定这些外部变量呢？Lambda表达式通过在最前面的方括号[]来明确指明其内部可以访问的外部变量，这一过程也称过Lambda表达式“捕获”了外部变量。

```
#include <iostream>
using namespace std;

int main()
{
    int a = 123;
    auto f = [a] { cout << a << endl; }; 
    f(); // 输出：123

    //或通过“函数体”后面的‘()’传入参数
    auto x = [](int a){cout << a << endl;}(123); 
}  
```

上面这个例子先声明了一个整型变量a，然后再创建Lambda表达式，该表达式“捕获”了a变量，这样在Lambda表达式函数体中就可以获得该变量的值。

类似参数传递方式（值传递、引入传递、指针传递），在Lambda表达式中，外部变量的捕获方式也有值捕获、引用捕获、隐式捕获。

### 1、值捕获

值捕获和参数传递中的值传递类似，被捕获的变量的值在Lambda表达式创建时通过值拷贝的方式传入，因此随后对该变量的修改不会影响影响Lambda表达式中的值。

示例如下：

```
int main()
{
    int a = 123;
    auto f = [a] { cout << a << endl; }; 
    a = 321;
    f(); // 输出：123
}  1 2 3 4 5 6 7 int main() {    int a = 123;    auto f = [a] { cout << a << endl; };     a = 321;    f(); // 输出：123 }   
```

***这里需要注意的是，如果以传值方式捕获外部变量，则在Lambda表达式函数体中不能修改该外部变量的值。\***

### 2、引用捕获

使用引用捕获一个外部变量，只需要在捕获列表变量前面加上一个引用说明符 `&`。如下：

```
int main()
{
    int a = 123;
    auto f = [&a] { cout << a << endl; }; 
    a = 321;
    f(); // 输出：321
}   
```

从示例中可以看出，引用捕获的变量使用的实际上就是该引用所绑定的对象。

### 3、隐式捕获

上面的值捕获和引用捕获都需要我们在捕获列表中显示列出Lambda表达式中使用的外部变量。除此之外，我们还可以让编译器根据函数体中的代码来推断需要捕获哪些变量，这种方式称之为隐式捕获。隐式捕获有两种方式，分别是`[=]`和`[&]`。`[=]`表示以值捕获的方式捕获外部变量，`[&]`表示以引用捕获的方式捕获外部变量。

隐式值捕获示例：

```
int main()
{
    int a = 123;
    auto f = [=] { cout << a << endl; };    // 值捕获
    f(); // 输出：123
}  
```

隐式引用捕获示例：

```
int main()
{
    int a = 123;
    auto f = [&] { cout << a << endl; };    // 引用捕获
    a = 321;
    f(); // 输出：321
}  
```

### 4、混合方式

上面的例子，要么是值捕获，要么是引用捕获，Lambda表达式还支持混合的方式捕获外部变量，这种方式主要是以上几种捕获方式的组合使用。

到这里，我们来总结一下：C++11中的Lambda表达式捕获外部变量主要有以下形式：

| 捕获形式    | 说明                                                         |
| :---------- | :----------------------------------------------------------- |
| []          | 不捕获任何外部变量                                           |
| [变量名, …] | 默认以值得形式捕获指定的多个外部变量（用逗号分隔），如果引用捕获，需要显示声明（使用`&` 说明符） |
| [this]      | 以值的形式捕获this指针                                       |
| [=]         | 以值的形式捕获所有外部变量                                   |
| [&]         | 以引用形式捕获所有外部变量                                   |
| [=, &x]     | 变量x以引用形式捕获，其余变量以传值形式捕获                  |
| [&, x]      | 变量x以值的形式捕获，其余变量以引用形式捕获                  |

## 修改捕获变量

前面我们提到过，在Lambda表达式中，如果以传值方式捕获外部变量，则函数体中不能修改该外部变量，否则会引发编译错误。那么有没有办法可以修改值捕获的外部变量呢？这是就需要使用mutable关键字，该关键字用以说明表达式体内的代码可以修改值捕获的变量，示例：

```
int main()
{
    int a = 123;
    auto f = [a]()mutable { cout << ++a; }; // 不会报错
    cout << a << endl; // 输出：123
    f(); // 输出：124
}   
```

## Lambda表达式的参数

Lambda表达式的参数和普通函数的参数类似，那么这里为什么还要拿出来说一下呢？原因是在Lambda表达式中传递参数还有一些限制，主要有以下几点：

- 1. 参数列表中不能有默认参数
- 1. 不支持可变参数
- 1. 所有参数必须有参数名

常用举例：

```
{
　　　　 int m = [](int x) { return [](int y) { return y * 2; }(x)+6; }(5);
        std::cout << "m:" << m << std::endl;            　　//输出m:16

        std::cout << "n:" << [](int x, int y) { return x + y; }(5, 4) << std::endl;            //输出n:9
        
        auto gFunc = [](int x) -> function<int(int)> { return [=](int y) { return x + y; }; };
        auto lFunc = gFunc(4);
        std::cout << lFunc(5) << std::endl;

        auto hFunc = [](const function<int(int)>& f, int z) { return f(z) + 1; };
        auto a = hFunc(gFunc(7), 8);

        int a = 111, b = 222;
        auto func = [=, &b]()mutable { a = 22; b = 333; std::cout << "a:" << a << " b:" << b << std::endl; };

        func();
        std::cout << "a:" << a << " b:" << b << std::endl;

        a = 333;
        auto func2 = [=, &a] { a = 444; std::cout << "a:" << a << " b:" << b << std::endl; };
        func2();

        auto func3 = [](int x) ->function<int(int)> { return [=](int y) { return x + y; }; };

    　　
　　　　 std::function<void(int x)> f_display_42 = [](int x) { print_num(x); };
	f_display_42(44);
　　}  
```

详细的使用方式可以参考`cppreference.com`的`Lambda 表达式 (C++11 起)`的文档说明。

## 参考

1. [C++ 11 Lambda表达式](https://www.cnblogs.com/DswCnblog/p/5629165.html)
2. [Lambda 表达式 (C++11 起)](https://zh.cppreference.com/w/cpp/language/lambda)



# C++ 阻止类被继承（继承类被实例化）的几种常用办法

不时会留意到有人问起如何阻止 C++ 中的类被继承，但多数人都没有把这个问题问对。

在 C++11 标准之前，阻止类被继承在语法上是做不到的，大家通常做到的只是`继承而来的类不能被实例化了`。这样一来，继承得到的类就完全没有用了。虽然最终的效果一致，但对问题的理解其实是有差异的。

从 C++11 标准开始，C++ 引入了一个新的关键字 `final`，只有被 `final` 修饰的类才能真正做到不能被继承。

下面举例实现常用的以达到“不能被继承/继承类不能被实例化”的几种手段。

## 私有化构造函数

```
class Demo1
{
public:
    static Demo1* Create()
    {
        return new Demo1;
    }

private:
    // 注意：私有掉构造函数
    Demo1() {}
    Demo1(const Demo1&);
    void operator=(const Demo1&);
};

class Test1 : public Demo1
{

};

int main()
{
//  Demo1*      pd1 = new Demo1;        // 错误：不能调用私有构造
    Demo1*      pd1 = Demo1::Create();
//  Test1*      pt1 = new Test1;        // 错误：不能调用基类私有构造
}
```

## 使用 虚继承 + 私有基类构造函数

```
template<typename T>
class SealedT
{
    // 注意：不是 friend class T;
    friend T;

private:
    SealedT() {}
};

// 注意：是虚继承
class Demo2 : virtual SealedT<Demo2>
{

};

int main()
{
    Demo2*      pd2 = new Demo2;
//  Test2*      pt2 = new Test2;        // 错误：不能访问间接虚基类
}  
```

## 使用 虚继承 + 受保护的基类构造函数

这个其实与前面一种形式是类似的。

```
class Sealed
{
protected:
    Sealed() {}
};

class Demo3 : virtual Sealed
{

};

class Test3 : public Demo3
{

};

int main()
{
    Demo3*      pb3 = new Demo3;
//  Test3*      pt3 = new Test3;        // 错误：不能访问间接虚基类
}  
 
```

## 使用 final 关键字

这个是最简单、最直接、最完美的。C++ 标准为啥如此晚才推出此功能。

```
class Demo4 final
{

};

// 不能从被 final 修饰的类继承，编译失败
/*
class Test4 : public Demo4
{

};
*/  

```

## 参考

1. [Prevent class inheritance in C++](https://stackoverflow.com/questions/2184133/prevent-class-inheritance-in-c)
2. [Virtual inheritance](https://en.wikipedia.org/wiki/Virtual_inheritance)



# Vector

## 初始化

(1): vector<int> ilist1;

默认初始化，vector为空， size为0，表明容器中没有元素，而且 capacity 也返回 0，意味着还没有分配内存空间。这种初始化方式适用于元素个数未知，需要在程序中动态添加的情况。

(2): vector<int> ilist2(ilist);

vector<int> ilist2  = ilist; 

两种方式等价 ，ilist2 初始化为ilist 的拷贝，ilist必须与ilist2 类型相同，也就是同为int的vector类型，ilist2将具有和ilist相同的容量和元素

(3): vector<int> ilist = {1,2,3.0,4,5,6,7};

 vector<int> ilist {1,2,3.0,4,5,6,7};

ilist 初始化为列表中元素的拷贝，列表中元素必须与ilist的元素类型相容，本例中必须是与整数类型相容的类型，整形会直接拷贝，其他类型会进行类型转换。

(4): vector<int> ilist3(ilist.begin()+2,ilist.end()-1);

ilist3初始化为两个迭代器指定范围中元素的拷贝，范围中的元素类型必须与ilist3 的元素类型相容，在本例中ilist3被初始化为{3,4,5,6}。注意：由于只要求范围中的元素类型与待初始化的容器的元素类型相容，因此迭代器来自不同的容器是可能的，例如，用一个double的list的范围来初始化ilist3是可行的。另外由于构造函数只是读取范围中的元素进行拷贝，因此使用普通迭代器还是const迭代器来指出范围并没有区别。这种初始化方法特别适合于获取一个序列的子序列。

(5): vector<int> ilist4(7);

默认值初始化，ilist4中将包含7个元素，每个元素进行缺省的值初始化，对于int，也就是被赋值为0，因此ilist4被初始化为包含7个0。当程序运行初期元素大致数量可预知，而元素的值需要动态获取的时候，可采用这种初始化方式。

(6):vector<int> ilist5(7,3);

指定值初始化，ilist5被初始化为包含7个值为3的int

## 排序

- sort(vec.rbegin(), vec.rend());

  //逆向排序 即按照从大到小的顺序进行排序   

- sort(vec.begin(), vec.end());

  //正向排序 即按照从小到大的顺序排序    

## 合并

​	vector<int> vec3;//vec3是空的
​    vec3.insert(vec3.end(),vec1.begin(),vec1.end())//将vec1压入
​    vec3.insert(vec3.end(),vec2.begin(),vec2.end())//继续将vec2压入
