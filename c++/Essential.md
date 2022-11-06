inline  --定义常被置于头文件中。

最适合声明为inline函数--体积小、常被调用，所从事的计算并不复杂。

防卫式声明

构造函数:

```c++

Book::Book(const string &title, const string &author) : _title(title), _author(author)
{
    cout << "Book::Book( " << _title << "," << _author << " ) constructor\n";
}

const string &title  ---不可改 
    参数传递尽量传引用
   string &title---可更改
```

参数传递和返回值传递---尽量  by  reference(&)

```c++
class complex
```

相同的class的各个object互为friend



定义一个类：

1. 数据一定放在private里，函数public--大多数
2. 参数尽可能以reference传--const视情况而定
3. 返回值尽量reference
4. 类内的函数 能加const 要加 const
5. Book::Book(const string &title, const string &author) : _title(title), _author(author)---“：” --一定要用。

什么情况下可以pass by reference

什么情况下可以return by reference---传递者无需知道接收者是以reference形式接受





操作符重载 _1---成员函数--this:







操作符重载 _2---非成员函数--this:





class with pointer members 必须有 拷贝构造  拷贝复制



```c++
//浅拷贝只是把指针拷贝过来---造成内存泄露
//拷贝构造--深拷贝
String::String(const String &str)
{
	m_data = new char[strlen(str.m_data)+1];
    strcpy(m_data,str.m_data);
}
```

检测自我复制

```c++
//拷贝复制函数
String & operator=(const String& str);  //和  &str 不同
{	
    if(this == &str)
        return *this;
    
    delete [] m_data;
	m_data = new char[strlen(str.m_data)+1];
    strcpy(m_data,str.m_data);
    return *this;
}
```





堆栈内存管理:





静态 static---无this point



静态变量  --必须在类外定义类内只是声明

```c++
class Account
{
	public:
		static double m_rate;
}

double Account::m_rate = 8.0;

int main()
{
    Account::set_rate(5.0);  //1 通过 object调用
    
    Account a;
    a.set_rate(7.0);		//2 通过class name  调用
}
```













模板函数(function template)

template <typename elemType>  typename--关键词，elemType--任意名称 也可以为-T等等

```c++
template<typename elemType>
void display_message(const string &msg, const vector<elemType> &vec);
```

函数指针

