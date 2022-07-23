### 第十二章	动态内存

程序用堆来存储动态分配的对象。

#### 12.1	动态内存与智能指针

**new,**在动态内存中为对象分配空间并返回一个指向该对象的指针，我们可以选择对象进行初始化；**delete**，接受一个动态对象的指针，销毁该对象，并释放与之关联的内存。

shared_ptr允许多个指针指向同一个对象；unique_ptr则“独占"所指的对象。标准库还定义了一个名为weak_ptr的伴随类，它是一种弱引用，指向shared_ptr所管理的对象。这三种类型都定义在memory头文件中。

##### 12.1.1	shared_ptr类

```c++
	shared_ptr<string> p1;			//shared_ptr,可以指向string
	shared_ptr<list<int>> p2;		//shared_ptr,可以指向int的list
```

**make_shared函数**

```c++
	//指向一个值为42的int的shared_ptr
	shared_ptr<int> p3 = make_shared<int>(42);
	//p4指向一个值为"99999999"的string
	shared_ptr<string> p4 = make_shared<string>(10,"9");
	//p5指向一个值初始化的int，即值为0
	shared_ptr<int> p5 = make_shared<int>();

	//常用auto定义一个对象来保存make_shared的结果
	auto p6 = make_shared<vector<string>>();
```

**shared_ptr的拷贝和赋值**

```c++
    auto p = make_shared<int>(42);  //p指向的int只有一个引用者
    auto q(p);                      //p和q指向相同的对象，此对象有两个引用者
    auto r = make_shared<int>(42);
    r = q;//给r赋值，令它指向另一个地址
          //递增q指向的对象的引用计数
          //递减r原来指向对象的引用计数
          //r原来指向的对象已没有引用者，会自动释放
```

**note:**

到底是用一个计数器还是其他数据结构来记录有多少指针共享对象，完全由标准库的具体实现来决定。关键是智能指针类能记录有多少个shared_ptr指向相同的对象，并能在恰当的时候自动释放对象。

**shared_ptr自动销毁所管理的对象......**

当指向一个对象的最后一个shared_ptr被销毁时，shared_ptr类会自动销毁此对象。它是通过另一个特殊的成员函数---**析构函数**完成销毁工作的。

shared_ptr的析构函数会递减它所指向的对象的引用计数。如果计数变为0，shared_ptr的析构函数就会销毁对象，并释放它占用的内存。

**......shared_ptr还会自动释放相关联的内存**

当动态对象不再被使用时，shared_ptr类会自动释放动态对象，这一特性使得动态内存的使用变得非常容易。

```c++
   shared_ptr<Foo> factory(T org)
    {
        //恰当处理arg
        //shared_ptr 负责释放内存
        return make_shared<Foo>(arg);
    }
  void use_factory(T arg)
    {
        shared_ptr<Foo> p = factory(arg);
        //使用p
        
    }//p离开了作用域，它指向的内存会被自动释放掉

  shared_ptr<Foo> use_factory(T arg)
    {
        shared_ptr<Foo> p = factory(arg);
        //使用p
        return p;   //当我们返回p时，引用计数进行了递增操作
    }//p离开了作用域，它指向的内存会被自动释放掉
```

**使用了动态生存期的资源的类**

程序使用动态内存三种原因

- 程序不知道自己需要使用多少对象
- 程序不知道所需对象的准确类型
- 程序需要再多个对象间共享数据

```c++
void PrintVector(vector<string> &vec)
{
    for (vector<string>::iterator it = vec.begin(); it != vec.end(); ++it)
    {
        cout << *it << "  ";
    }
    cout << endl;
}
vector<string> v1;
    {
        vector<string> v2 = {"a", "an", "the"};
        v1 = v2;				//从v2拷贝元素到v1中
        cout << "v1" << endl;	
        PrintVector(v1);		
        cout << "v2" << endl;
        PrintVector(v2);
        //v2被销毁，其中的元素也被销毁
        //v1有三个元素，时原来v2中元素的拷贝
    }//由一个vector分配的元素只有当这个vector存在时才存在。当一个vector被销毁时，这个vector中的元素也都被销毁
```

```c++
Blob<string> b1;//空Blob
{
    //新作用域
    Blob<string> b2 = {"a","an","the"};
    b1 = b2;	//b1和b2共享相同的元素
    //b2被销毁了，但是b2中的元素不能被销毁
    //b1指向最初由b2创建的元素
}
```

**定义StrBlob类**

**StrBlob构造函数**

**元素访问成员函数**

##### 12.1.2直接管理内存

**使用new动态分配和初始化对象**

在自由空间分配的内存是无名的，因此new无法为其分配对象命名，而是返回一个指向该对象的指针。

```c++
    int *pi = new int;  //pi指向一个动态分配、未初始化的无名对象。
    string *ps = new string;//初始化为空的string
    int *pi = new int;      //pi指向一个未初始化的int

    int *pi = new int(1024);            //pi指向的对象的值为1025    
    string *ps = new string(10, '9');   //*ps为"9999999999"

    //vector由10个元素，值依次从0到9
    vector<int> *pv = new vector<int>{0, 1, 2, 3, 4, 5, 6, 7, 8, 9};
```

也可以对动态分配的对象进行值初始化，只需在类型名之后跟一对空括号即可：

```c++
    string *p1 = new string;       //默认初始化为空string
    string *ps = new string();     //默认初始化为空string
    int *pi1 = new int;            //默认初始化，*pi1值未定义
    int *pi2 = new int();           //值初始化为0；*pi2为0
```

**出于与变量初始化相同的原因，对动态分配的对象进行初始化通常是个好主意。**

由于编译器要用初始化器的类型来推断要分配的类型，只有当括号中仅有单一初始化器时才可以使用**auto**

```c++
auto p1 = new auto(obj);    //p指向一个与obj类型相同的对象
                                //该对象用obj进行初始化
auto p2 = new auto{a, b, d};    //错误：括号内只能有单个初始化器
```

**动态分配的const对象**

用new分配const对象是合法的：

```c++
//分配并初始化一个const int
const int *pci = new const int(1024);
//分配并默认初始化一个const的空string
const string *pcs = new consr string;
```

一个动态分配的const对象必须进行初始化，对于一个定义了默认构造函数的类类型，其const动态对象可以隐式初始化，而其它类型的对象必须显示初始化。由于分配对象是const的，new返回的指针是一个指向const的指针。

**内存耗尽**

一旦一个程序用光了它所要求的内存空间，他会抛出一个类型为bad_alloc的异常。我们可以改变使用new的方式阻止它抛出异常：

```c++
    //如果分配失败
    int *p1 = new int;          //如果分配失败，new抛出bad_alloc
    int *p2 = new (nothrow) int;//如果分配失败，new返回一个空指针
//我们称这种形式的的new为--定位new，
```

**释放动态内存**

为了防止内存耗尽，在动态内存使用完毕后，必须将其归还给系统。我们通过**delete表达式**来将动态内存归还给系统。delete表达式接受一个指针，指向我们想要释放的对象：

```c++
delete p;	//p必须指向一个动态分配的对象或是一个空指针
//两个动作 销毁给定的指针指向的对象；释放对应的内存
```

**指针值和delete**

我们传递给delete的指针必须指向动态分配的内存，或者是一个空指针。释放一块并非new分配的内存，或者将相同的指针值释放多次其行为是未定义的。

```c++
  int i, *pi1 = &i, *pi2 = nullptr;
    double *pd = new double(33), *pd2 = pd;
    delete i;      	 //错误： i不是一个指针
    delete pi1;      //未定义：pi1指向一个局部变量
    delete pd;       //正确
    delete pd2;      //未定义：p2指向的内存已经被释放
    delete pi2;      //正确：释放一个空指针总是没有错误的
```

```c++
    const int *pci = new const int(1024);
    delete pci;     //正确：释放一个const对象
```

虽然一个const对象的值不能改变，但他的本身是可以被销毁的。如同任何其他动态对象一样，想要释放一个const动态对象，只要delete指向它的指针即可。

**动态对象的生存期直到被释放时为止**







注：由内置指针管理的动态内存在被显示释放前一直都会存在。

**动态内存的管理非常容易出错**

- 忘记delete内存。忘记释放动态内存会导致人们常说的"内存泄漏"问题，因为这种内存永远不可能被归还给自由空间了，查找内存泄漏错误是非常困难的，因为通常应用程序运行很长时间后，真正耗尽内存时，才能检测到这种错误。
- 使用已经释放掉的对象。通过在释放内存后将指针置为空，有时可以检测出这种错误。
- 同一块内存释放两次。当有两个指针指向相同的动态分配对象时，可能发生这种错误。如果对其中一个指针进行了delete操作，对象的内存就被归还给自由空间了。如果我们随后又delete第二个指针，自由空间就可能被破坏。
- 坚持只使用智能指针，就可以避免所有这些问题。对于一块内存，只有在没有任何智能指针指向它的情况下，智能指针才会自动释放它。

**delete之后重置指针值........**

在delete之后，指针就变成了人们所说的**空悬指针**，即，指向一块曾保存数据对象但现在已经无效的内存指针。

未初始化指针的所有缺点空悬指针也都有。有一种方法可以避免空悬指针的问题：在指针即将要离开其作用域之前释放掉它所关联的内存。

**.......这只是提供了有限的保护**

动态内存的一个基本问题是可能有多个指针指向相同的内存。在delete内存之后重置指针的方法只对这个指针有效，对其他任何仍指向(已释放)内存的指针是没有作用的。

```c++
 int *p(new int(42));    //p指向动态内存
    auto q = p;             //p和q指向相同的内存
    delete p;               //p和 q均变为无效
    p = nullptr;            //指出p不在绑定到任何对象
```

##### 12.1.3 shared_ptr和new结合使用

```c++
    //我们还可以用new返回的指针初始化智能指针
    shared_ptr<double> p1;  //shared_ptr可以指向一个double
    shared_ptr<int> p2(new int(42));    //p2指向一个值为42的int
```

接受指针参数的智能指针构造函数是explicit的。因此，我们不能将一个内置指针隐式转换为一个智能指针，必须使用直接初始化形式来初始化一个智能指针：

```c++
    shared_ptr<int> p1 = new int(33);   //错误：必须使用直接初始化形式
	//p1的初始化隐式的要求编译器用一个new返回的int* 来创建一个shared_ptr。由于我们不能进行内置指针到智能指针间的隐式转换，因此这条语句是错误的
    shared_ptr<int> p2(new int(22));    //正确:使用了直接初始化形式
```

类似，一个返回shared_ptr的函数不能在其返回语句中隐式转换一个普通指针：

```c++
shared_ptr<int> clone(int p)
{
    return new int(p);  //错误：隐式转换为shared_ptr<int>
}
```

```c++
shared_ptr<int> clone(int p)
{
    return shared_ptr<int> (new int(p));  //正确：显示的用int*创建shared_ptr<int>
}
```

**不要混合使用普通指针和智能指针.......**

shared_ptr可以协调对象的析构，但这仅局限于其自身的拷贝之间。这也是我们推荐使用make_shared而不是new的原因。

> 注：使用一个内置指针来访问一个智能指针所负责的对象是很危险的，因为我们无法知道对象何时会被销毁

**......也不要用get初始化另一个智能指针或为智能指针赋值**

智能指针类型定义了一个名为get的函数，它返回一个内置指针，指向智能指针管理的对象。

此函数就是为这样的一种情况设计的：我们需要向不能使用智能指针的代码传递一个内置指针。使用get返回的指针的代码不能delete此指针。

> warning：get用来将指针的访问权限传递给代码，你只有在确定代码不会delete指针的情况下，才能使用get。特别是，永远不要用get初始化另一个智能指针或者为另一个智能指针赋值。

**其他share_ptr操作**

```c++
//我们可以用reset来将一个新的指针赋予一个shared_ptr: 
p = new int(1024);			//错误:不能将一个指针赋予shared_ptr
p.reset(new int(1024));		//正确：p指向一个新对象
```

与赋值类似，reset会更新引用计数，如果需要的话，会释放p指向的对象。reset成员经常与unique一起使用，来控制多个shared_ptr共享的对象。

```c++
if(!p.unique())
{
    p.reset(new string(*p));	//我们不是唯一用用户；分配新的拷贝
}
*p += newVal;	//现在我们知道自己是唯一用户，可以改变对象的值
```

##### 12.1.4智能指针和异常

当发生异常时，我们直接管理的内存是不会自动释放的。如果使用内置指针管理内存，且在new之后在对应的delete之前发生了异常，则内存不会被释放：

```c++
void f()
{
    int *ip = new int(42);  //动态分配一个新对象
    //这段代码抛出一个异常，且在f中未被捕获
    delete ip;  //退出之前释放内存

}
//如果在new和delete之间发生异常，且异常未在f中被捕获，则内存就永远不会被释放了。在函数f之外没有指针指向这块内存，因此就无法释放它
```

**智能指针和哑类**

**使用我们自己的释放出操作**

> 智能指针陷阱：
>
> - 不使用相同的内置指针值初始化（或）reset多个智能指针
> - 不delete get()    返回的指针
> - 如果你使用get() 返回的指针，记住当最后一个对应的智能指针销毁后，你的指针就变为无效了。
> - 如果你使用智能指针管理的资源部不是new分配的内存，记住传递给它一个删除器。

##### 12.1.5	unique_ptr

定义一个unique_ptr时，需要将其绑定到一个new返回的指针上。类似shared_ptr,初始化unique_ptr必须采用**直接初始化方式**

```c++
    unique_ptr<double> p1;              //可以指向一个double的unique_ptr
    unique_ptr<int> p2(new int(42));    //p2指向一个值为42的int
```



```c++
 //由于unique_ptr拥有他指向的对象，因此unique_ptr不支持普通拷贝或赋值操作
    unique_ptr<string> p1(new string("Stegosaur")); 
    unique_ptr<string> p2(p1);              //错误：unique_ptr不支持拷贝
    unique_ptr<string> p3;
    p3 = p2;                                //错误：unique_ptr不支持赋值
```

虽然我们不能拷贝或赋值unique_ptr，但可以通过调用release或reset将指针的所有权从一个（非 const）unique_ptr转移给另一个unique：

```c++
    //将所有权从P1转移给p2
    unique_ptr<string> p2(p1.release());		//release将p1置为空
    unique_ptr<string> p3(new string("Trex"));
    //将所有权从p3转移给p2
    p2.reset(p3.release());						//reset释放了p2原来指向的内存
```

**传递unique_ptr参数和返回unique_ptr**

不能拷贝unique_ptr的规则有一个例外：我们可以拷贝和赋值一个将要被销毁的unique_ptr

**向unique_ptr传递删除容器**

##### 12.1.6	weak_ptr

将一个weak_ptr绑定到一个shared_ptr不会改变shared_ptr的引用计数。一旦最后一个指向对象的shared_ptr被销毁，对象就会被释放。即使有weak_ptr指向对象，对象也还是会被释放，因此，weak_ptr的名字抓住了这种智能指针"弱"共享对象的特点。

当我们创建创建一个weak_ptr时，要用一个shared_ptr来初始化它：

```c++
auto p = make_shared<int>(42);
weak_ptr<int> wp(p);	//wp弱共享p；p的引用计数未改变

```

由于对象可能不存在，我们不能使用weak_ptr直接访问对象，而必须调用lock。

**核查指针类**

**指针操作**

#### 12.2动态数组

##### 12.2.1	new和数组

为了让new分配一个对象数组，我们要在类型名后面跟一对方括号，在其中指明要分配的对象的数目。

```c++
int *pia = new int[get_size()]; // pia指向第一个int
//方括号中的大小必须是整型，但不必是常量。
```

**分配一个数组会得到一个元素类型的指针**

当使用new分配一个数组时，我们并未得到一个数组类型的对象，而是得到一个数组元素类型的指针。

> 要记住我们所说的动态数组并不是数组类型，这是很重要的

**初始化动态分配对象的数组**

```c++
    int *pia = new int[10];      	 	  //10个未初始化的int
    int *pia2 = new int[10]();    		  //10个值初始化为0的int
    string *psa = new string[10];  		 //10个空string
    string *psa2 = new string[10]();    //10个空string

	//还可以用提供一个元素初始化器的花括号列表：
    //10个int分别用列表中对应的初始化器初始化
    int *pia3 = new int[10]{0, 1, 2, 3, 4, 5, 6, 7, 8, 9};
    //10个string，前4个用给定的容器初始化，剩余的进行值初始化
    string *psa3 = new string[10] { "a", "an", "the", string(3, 'x') };
	//如果初始化器数目小于元素数目，剩余元素进行值初始化
	//如果初始化器数目大于元素数目，则new表达式失败，不会分配任何内存。
```

**动态分配一个空数组是合法的**

```c++
    char arr[0]; //错误:不能定义长度为0的数组
    char *cp = new char[0];//正确:但cp不能被解引用
```

**释放动态数组**

```c++
delete p;			//p必须指向一个动态分配的对象或为空
delete [] pa;		//pa必须指向一个动态分配的数组或为空
//数组逆序被销毁，最后一个先被销毁。。
//当我们释放一个指向数组的指针时，空括号对是必须的：
```

**智能指针和动态数组**

```c++
   //up指向一个包含10个未初始化的int数组
    unique_ptr<int[]> up(new int[10]);  
    up.release();       //自动调用delete销毁其指针
```

shared_ptr不直接支持管理动态数组，如果使用其来管理动态数组，必须提供自己定义的删除器。

```c++
  //为了使用shared_ptr必须提供一个删除器
    shared_ptr<int> sp(new int[10], [](int *p)
                       { delete[] p; });
    sp.reset();         //使用我们提供的lambda释放数组,它使用delete[]


    //shared_ptr未定运算符,并且不支持指针的算术运算
    for (size_t i = 0; i < 10; ++i)     
    {
        *(sp.get() + i) = i;    //使用get获取一个内置指针
    }
    //为了访问数组中的元素，必须用get获取一个内置指针，然后用它来访问数组元素。
```

##### 12.2.2	allocator类

 标准库allocator类定义在头文件memory中，它帮助我们将内存分配和对象构造分离开来。

```c++
    allocator<string> alloc;        //可以分配string的allocator对象
    auto const p = alloc.allocate(n);   //分配n个未初始化的string
```

**allocator 分配未构造的内存**

allocator 分配的内存是未构造的。我们按需要在此内存中构造对象。

construct成员函数接受一个指针和零个或多个额外参数，在给定位置构造一个元素。额外参数用来初始化构造的对象。

```c++

    auto q = p; //q指向最后构造的元素之后的位置
    alloc.construct(q++);//*q为空字符串
    alloc.construct(q++, 10, 'c');  //*q为cccccccc
    alloc.construct(q++, "hi");     //*q为hi

	cout << *p <<endl; //正确：使用string的输出运算符
	cout << *q <<endl; //错误：q指向未构造的内存
```

> 为了使用allocate返回的内存，我们必须用construct构造对象。使用未构造的内存，其行为是未定义的。

**拷贝和填充未初始化内存的算法**

```c++
    //分配比vi中元素所占空间大一倍的动态内存
    allocator<int> alloc;

    auto p = alloc.allocate(v.size() * 2);
    //通过拷贝vi中元素来构造从p开始的元素
    auto q = uninitialized_copy(vi.begin(), vi.end(), p);
    //将剩余元素初始化为42
    uninitialized_fill_n(q, vi.size(), 42);
```

uninitialized_copy返回（递增后的）目的位置迭代器。一次uninitialized_copy调用后会返回一个指针其指向最后一个构造的元素之后的位置。

#### 12.3	使用标准库： 文本查询程序
