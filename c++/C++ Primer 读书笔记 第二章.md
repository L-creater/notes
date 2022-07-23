## 2 变量和基本类型

### 2.2 变量

**列表初始化**

```c++
	long double ld = 3.1415926536;
	int a{ ld }, b = { ld };	//错误：转换未执行，因为存在丢失信息的危险
	int c(ld), d = ld;			//正确：转换执行，且确实丢失了部分值
	//注意，当使用{}的形式来初始化变量，如果存在丢失信息的风险时，编译器会报错，其它形式有警告，但不报错。
```

#### 2.2.2 变量声明和定义的关系

```c++
//如果想声明一个变量而非定义，就在变量名前添加关键字extern，而且不要显示的初始化变量：
extern int i;//声明i而非定义i
int j;		 //声明并定义j
```

**嵌套的作用域**

作用于能彼此包含，被包含的的作用域（或者说被嵌套的）称为内层作用域，包含着别的作用域的作用域称为外层作用域。

作用域种一旦声明了某个名字，他所嵌套着的所有作用域种都能访问改名字。同时，允许在内层作用域中重新定义外层作用域已有的的名字：

```c++
#include<iostream>
using namespace std;

int reused = 42;//reused拥有全局作用域
int main()
{
	int unique = 0; //unqiue拥有块作用域
	// 输出#1：全局变量reused；输出 42 0
	cout << reused << " " << unique << endl;
	int reused = 0; //新建的局部变量reused ， 覆盖了reused
	//输出#2：使用局部变量reused；输出 0 0
	cout << reused << unique << endl;
	//输出#3：显示地访问全局变量reused; 输出 42 0
	cout << ::reused << " " << unique << endl;
	return 0;
}
```

#### **2.3.1 引用**

**引用必须初始化**

```c++
int ival = 1024;
int &refval = ival;
int &refVal2;//报错：引用必须初始化
```

**引用即别名**

应用只能绑定在对象上，而不能与字面值或某个表达式的计算结果绑定在一起。

```c++
	int ival = 1;
	int& refval = 1;	//错误
	const int& refval = 1;	//正确： refval是一个常量引用
	int& refval2 = ival;

```

#### 2.3.2 指针

其他所有指针的类型都要和它所指向的对象严格匹配

建议：初始化所有指针

```c++
int i = 42,j=20;
	int* pi = &i;						//定义一个指针变量指向i,即pi保存的是i的地址
	*pi=j;								//把j赋值给pi指向的变量，因为pi指向i所以相当于把j赋值给i   
	cout << "*pi： " << *pi << endl;
	cout << "i: " << i << endl;
```

**void* 指针**--------

void* 可以存放任意对象的地址。

```c++
	double obj = 3.14, * pd = &obj;
									//void* 能存放任意类型对象的地址
	void* pv = &obj;				//obj可以是任意类型的对象
	pv = pd;						//pv可以存放任意类型的指针
```

#### 2.4.1 const的引用

**对const的引用可能引用一个并非const的对象**

常量引用仅对引用可参与的操作做出了限定，对于引用的对象本身是不是一个常量未作限定，因为对象也可能是一个非常量，所以允许通过其他途径改变它的值。

```c++
	int i = 42;
	int& r1 = i;
	const int& r2 = i;
	const int& r3 = r1;
	r1 = 0;
	r2 = 0;     //错误：r2是一个常量引用
```

#### 2.4.2 指针和const

```c++
	const double pi = 3.14;
	double* ptr = &pi;		//ptr是一个普通指针 
	const double* cptr = &pi;
	*cptr = 33;				//错误：不能给*cptr赋值
```

#### 2.4.3 顶层const

顶层const表示指针本身是个常量，底层const表示指针所指向的对象是一个常量。

```c++
int i = 0;	
int* const p1 = &i;		//不能改变p1的值，顶层const
const int ci = 22;		//不能改变ci的值，这是一个顶层const
const int* p2 = &ci;	//允许改变p2的值，底层const
const int* const p3 = &ci;//靠右的是顶层const,靠左的是底层const
const int& r = ci;		 //用于声明引用的const都是底层const
```

#### 2.4.4 constexpr和常量表达式

**常量表达式**是指值不会改变并且在编译过程中就能得到计算结果的表达式。

一个对象（或表达式）是不是常量表达式由它的数据类型和初始值共同决定：

```c++
const int max_files = 20;		//max_files是常量表达式
const int limit = max_files + 1;//limit是常量表达式
int staff_size = 27;			//不是	
const int sz = get_size();		//不是

```

**constexpr变量**

声明为constexpr的变量一定是一个常量，而且必须用常量表达式初始化

```c++

	constexpr int mf = 20;			//20是常量表达式
	constexpr int limit = mf + 1;	//mf+1是常量表达式
	constexpr int sz = size();		//只有当size是一个constexpr函数时，才是一条正确的声明语句
```
