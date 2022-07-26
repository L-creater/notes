### 第十三章	拷贝控制

当定义一个类时，我们显示的或隐式的指定在此类型的对象拷贝、移动、赋值、销毁时做什么。一个类通过定义五种特殊成员函数来控制这些操作，包括：**拷贝构造函数、拷贝赋值运算符、移动构造函数、移动赋值运算符、析构函数**

#### 13.1	拷贝、赋值与销毁

##### 13.1.1	拷贝构造函数

如果一个构造函数的第一个参数是自身类类型的引用，且任何额外参数都有默认值，则此构造函数是拷贝构造函数。

```c++
class	Foo
{
	Foo();				//构造函数
	Foo(const Foo&);	//拷贝构造函数
}
```

**合成拷贝构造函数**

若果我们没有为一个类定义一个拷贝构造函数，编译器会为我们定义一个，与合成默认构造函数不同，即使我们定义了其它构造函数，编译器也会为我们合成一个拷贝构造函数。

对某些类来说，**合成拷贝构造函数**用来阻止我们拷贝该类类型的对象。而一般默认情况，合成拷贝构造函数会将其参数的成员逐个拷贝到正在创建的对象中。





**拷贝初始化**

拷贝初始化通常使用拷贝构造函数来完成，或者依靠移动构造函数来完成。

拷贝初始化不仅仅在我们使用 = 定义变量时发生，在下列情况下也会发生：

- 将一个对象作为实参传递给一个非引用了类型的形参
- 从一个返回类型为非引用类型的函数返回一个对象
- 用花括号初始化一个数组中的元素或一个聚合类中的成员
- 某些类类型还会对他们所分配的对象使用拷贝初始化。 

**参数和返回值**

拷贝构造函数被用来初始化非引用类类型，这一**特性解释了为什么拷贝构造函数自己的参数必须是引用类型**。如果参数不是引用类型，则调用永远无法成功----为了调用拷贝构造函数，我们必须拷贝他的实参，但为了拷贝实参，我们有需要用拷贝构造函数，如此无限循环。

**拷贝初始化限制**

 如果我们使用的初始化值要求通过一个explicit的构造函数来进行类型转换，那么使用拷贝初始化还是直接初始化**就不是无**关紧要了

```c++
vector<int> v1(10);		//正确：直接初始化
vector<int> v2 = 20;	//错误：就是大小参数的构造函数是explicit的

void f(vectro<int>);	//f的参数进行拷贝初始化
f(10);					//错误：不能用一个explicit的构造函数拷贝一个实参
f(vector<int>(10));		//正确： 从一个int直接构造一个临时的vector 

//v2是错误的，因为vector接受的单一的大小参数是构造函数explicit的。当传递一个实参或从函数返回一个值是，我们不能隐式使用一个explicit构造函数。如果我们希望使用一个explicit构造函数，就必须显式的使用，像此代码最后一行那样。
```

**编译器可以绕过拷贝构造函数**

##### 13.1.2	拷贝赋值运算符

**重载赋值运算符**

重载运算符的参数表示运算符的运算对象。某些运算符，包括赋值运算符，必须定义为成员函数。如果一个运算符是一个成员函数，其左侧运算对象就绑定到隐式的this参数。对于一个二元运算符，其右侧运算对象作为显示参数传递。

> 赋值运算符通常应该返回一个指向其左侧运算对象的引用。

**合成拷贝构造赋值运算符**





##### 13.1.3	析构函数

 **析构函数完成什么工作**

在一个析构函数中首先执行函数体，然后销毁成员。成员按初始化顺序的逆序销毁。通常析构函数释放对象在生存期分配的所有资源。

在一个析构函数中，不存在类似构造函数中初始化列表的东西来控制成员如何销毁，析构部分是隐式的。成员销毁时发生什么完全依赖于成员的类型。销毁类类型的成员需要执行成员自己的析构函数。内置类型没有析构函数，因此销毁内置类型什么也不要做。

> 隐式销毁一个内置的智能指针类型的成员不会delete它所指向的对象

**什么时候调用析构函数**

无论何时一个对象被销毁，就会自动调用其析构函数：

- 变量在离开其作用域时被销毁。
- 当一个对象被销毁时，其成员被销毁
- 容器被销毁是，其元素被销毁
- 对于动态分配的对象，当对指向它的指针应用delete运算符时被销毁
- 对于临时对象，当创建他的完整表达式结束时被销毁。

**合成析构函数**

当一个类未定义自己的析构函数时，编译器会为它定义一个合成析构函数

**对于某些情况，合成的析构函数会被用来阻止该类型的对象被销毁**。如果不是这种情况，合成的析构函数的函数体就为空。

> 析构函数体自身并不直接销毁成员。成员是在析构函数体之后隐含的析构阶段中被销毁的。在整个对象销毁的过程中，析构函数体是作为成员销毁步骤之外的另一部分而进行的。



##### 13.1.4	三/五法则

**需要析构函数的类也需要拷贝和赋值操作**

当我们决定一个类是否要定义他自己的版本的拷贝控制成员时，一个基本原则是首先确定这个类是否需要一个析构函数。通常，对析构函数的需求要比对拷贝构造函数或赋值运算符的需求更为明显。如果这个类需要一个析构函数，我们几乎可以肯定他也需要一个拷贝构造函数和一个拷贝赋值运算符。

**需要拷贝操作的类也需要赋值操作**，**反之亦然**

虽然很多类需要定义所有拷贝控制成员，但某些类所要完成的工作，只需拷贝或赋值操作，不需要析构函数。



##### 13.1.5	使用=default

我们可以通过拷贝控制成员定义为=default来显示的要求编译器生成合成的 版本

```c++
class Sales_data
{
  public:
    Sales_data() = default;	//拷贝控制成员；使用default
    //当我们在类内使用=default修饰成员的声明时，合成的函数将隐式的声明为内联的
    //如果我们不希望合成的成员是内联函数，应该只对成员的类外定义使用=default，
};
```



##### 13.1.6	阻止拷贝

> 大多数类应该定义默认构造函数、拷贝构造函数和拷贝赋值运算符，无论是显示的还是隐式的。

**定义删除的函数**

我们可以通过将拷贝构造函数和拷贝赋值运算符定义为**删除的函数**

**删除的函数：**我们虽然声明了它们，但不能以任何方式使用它们，在函数的参数列表后面加上=delete来指出我们希望将他定义为删除的：

```c++
struct NoCopy
{
    NoCopy()=default;							//使用合成的默认构造函数
    NoCopy(const NoCopy&) = delete;				//阻止拷贝
    NoCopy &operator=(const NoCopy&) = delete;	//阻止赋值
    
}
```

 **析构函数不能是删除的成员** 

**合成的拷贝控制成员可能是删除的**

对某些类来说，编译器将这些合成的成员定义为删除的函数：

- 如果类的某个成员的析构函数是删除的或不可访问的（例如是  private），则类的合成析构函数被定义为删除。





> 本质上这些规则含义是：如果一个类有数据成员不能默认构造、拷贝、复制或销毁，则对应的成员函数将被定义为删除的。

**private控制拷贝**

#### 13.2	拷贝控制和资源管理

##### 13.2.1	行为像值的类

   





### 第十四章	重载运算与类型转换

#### 14.1	基本概念

重载运算符是具有特殊名字的函数：他们的名字由关键字operator和其后要定义的运算符共同构成。和其他函数一样，重载的运算符也包含返回类型、参数列表以及函数体。



> 当一个重载的运算符是成员函数时，this绑定到左侧运算对象。成员运算符函数的（显示）参数数量比运算对象的数量少一个。

**直接调用一个重载的运算符函数**





**某些运算符不应该被重载**

> 通常情况下，不应该重载逗号、取地址、逻辑与、逻辑或运算符

**使用与内置类型一致的含义**

-  如果类执行IO操作，则定义位移运算符使其与内置类型的IO类型保持一致。
- 如果类的某个操作是检查相等性，则定义operator==；如果类有了operator==，意味着它通常也应该有operator！=
- 如果类包含一个内在的单序比较操作，则定义operator<；如果类有了此操作，则它通常也有其他关系操作。
- 重载运算符的返回类型通常情况下与其内置版本的返回类型兼容：逻辑运算符和关系运算符应该返回bool，算术运算符应该返回一个类类型，赋值运算符和复合赋值运算符则应该返回左侧运算对象的一个引用。

**复制和复合赋值运算符**

**选择作为成员或者非成员**

- 赋值(=)、下标([ ])、调用（( )）、成员访问箭头（->）运算符必须是成员。
- 复合赋值运算符一般来说应该是成员，但非必须，这一点与赋值运算符略有不同。
- 改变对象状态的运算符或者与给定类型密切相关的运算符，如递增、递减、和解引用运算符，通常应该是成员。
- 具有对称性的运算符可能转换任意一端的运算对象，例如算术、相等性、关系、和位运算符等，因此他们通常应该是普通的非成员函数。

 

#### 14.2	输入和输出运算符



##### 14.2.1	重载输出运算符<<

通常情况下，输出运算符的第一个形参是一个非常量ostream对象的引用。之所以ostream是非常量是因为向流写入内容会改变其状态；而该形参是引用是因为我们无法直接复制一个ostream对象。

第二个形参一般来说是一个常量引用，该常量是我们想要打印的类类型。第二个形参是引用的原因是我们希望避免复制实参；而之所以该形参是常量是因为（通常情况下）打印对象不会改变对象的值。



**输出运算符尽量减少格式化操作**







**输入输出运算符必须是非成员函数**



输入输出运算符必须是非成员函数而不能是类的成员函数，否则，他的左侧对象将是我们的类的一个对象：

##### 14.2.2	重载输入运算符>>



#### 14.3	算术和关系运算符

 通常情况下，我们把算术和关系运算符定义成非成员函数以允许对左侧或右侧的运算对象进行转换。因为这些运算符一般不需要改变运算对象的状态，所以形参都是常量的引用。

##### 14.3.1	相等运算符

- 如果一个类含有判断两个对象是否相等的操作，则它显然应该把函数定义成operator==而非一个普通的命名函数：因为用户肯定希望能使用==比较对象，所以提供了==就意味着用户无需再费时费力的学习和记忆一个全新的函数名字。此外，类定义了==运算符后也更容易使用标准库容器和算法。
- 如果类定义了operator==，则运算符应该能判断一组给定的对象是否含有重复数据。

##### 14.3.2	关系运算符

通常情况下关系运算符应该：

1. 定义顺序关系，令其与关联容器中对关键字的要求一致，并且
2. 如果类同时也含有==运算符的话，则定义一种关系令其与==保持一致。特别的是，如果两个对象是！=的，那么一个对象应该< 另外一个

> 如果存在唯一一种逻辑可靠的<定义，则应该考虑为这个类定义<运算符。如果类同时还包含==，则当且仅当<的定义和==产生的结果一致时才定义<运算符。

#### 14.4	赋值运算符

 

```c++
StrVec &StrVec::operator=(initializer_list<string> il)
{
    //alloc_n_copy分配内存空间并从给定范围内拷贝元素
    auto data = alloc_n_copy(il.begin(), il.end());
    free();                 //销毁对象中的元素并释放内存空间
    elements = data.first;  //更新数据成员使其指向新空间
    first_free = cap = data.second;
    return *this;
}
```

> 我们可以重载赋值运算符，不论形参是什么，赋值运算符都必须定义为成员函数。

**复合赋值运算符**

> 赋值运算符必须定义为类的成员，复合赋值运算符通常情况下也应该这样做。这两类运算符都应该返回左侧运算对象的引用。

#### 14.5	下标运算符

> 下标运算符必须是成员函数。

下标运算符通常以所访问的元素的引用作为返回值。这样做的好处是下标可以出现在赋值运算符的任意一端。

> 如果一个类包含下标运算符，则他通常会定义两个版本：一个版本返回普通引用，另一个是类的常量成员并且返回常量引用。

```c++
class StrVec
{
private:
    string *elements;

public:
    string &operator[](size_t n)
    {
        return elements[n];
    }

    const string &operator[](size_t n) const
    {
        return elements[n];
    }
    StrVec(/* args */);
    ~StrVec();
};
```

####  14.6	递增和递减运算符

>  为了与内置版本保持一致，前置运算符应该返回递增或递减后对象的引用。
>
> 后置运算符应该返回对象的原值（递增或递减之前的值）,返回的形式是一个值而非引用

#### 14.7	成员访问运算符

#### 14.8	函数调用运算符  

​     

**含有状态的函数对象类**

##### 14.8.1	lambda是函数对象  

**表示lambda及相应捕获行为的类**

##### 14.8.2	标准库定义的函数对象

 **在算法中使用标准库函数对象**

##### 14.8.3	可调用对象与function

c++语言中有几种可调用对象：函数、函数指针、lambda表达式、bind创建的对象以及重载了函数调用运算符的类。

**不同类型可能具有相同的调用形式**

**标准库function类型**

**重载函数与function**

我们不能直接将重载函数的名字存入function类型的对象中。

消除二义性问题的一条途径是存储指针而非函数的名字，也可以使用lambda来消除二异性。

#### 14.9	重载、类型转换与运算符

##### 14.9.1	类型转换运算符

类型转换运算符是类的一种特殊成员函数，它负责将一个类类型的值转换为其它类型。



类型转换运算符既没有显示的返回类型，也没有形参，而且必须定义成类的成员函数。

**显示的类型转换运算符**

与显示的构造函数一样，编译器也不会将一个显示的类型转换运算符用于隐式类型转换。



**转换为bool**

##### 14.9.2	避免有二义性的类型转换

如果类中包含一个或多个类型转换，必须确保在类类型和目标类型之间存在唯一的转换方式。否则的话，我们编写的代码将很可能具有二义性。

**二义性与转换目标为内置类型的多重类型转换**

> 当我们使用两个用户定义类型转换时，如果转换函数之前或之后存在标准类型转换，则标准类型转换将决定最佳匹配到底是哪个。

**重载函数与转换构造函数**



**重载函数与用户定义的类型转换**

当调用重载函数时，如果两个用户定义的类型转换都提供了可行的匹配，则我们认为这些类型转换一样好。

##### 14.9.3	函数匹配与重载运算符

