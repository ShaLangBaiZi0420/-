oop8-9章

8-构造和析构

### 构造函数

创建对象的同时，进行初始化工作

#### 格式

名字与类名相同
无返回值
explicit关键字可选
有this，非static，非const

#### 自定义构造函数

可重载
可设置不同访问控制
可选explicit关键字
可带缺省参数(注意二义性问题)
只声明，无实现

#### 私有构造函数

(构造单例)

使用指针

```cpp
#include <iostream>
using namespace std;

class Name
{
public:
	static Name* creat(int age)
	{
		static Name* pObj = nullptr;
		if (pObj == nullptr)
			pObj = new Name(age);
		return pObj;
	}
	int getAge()
	{
		return m_age;
	}
private:
	Name(int age) 
	{
		this->m_age = age;
	}
	int m_age;
};

int main()
{
	Name* p1 = Name::creat(20);
	Name* p2 = Name::creat(25);
	cout << "p1 age: " << p1->getAge() << endl;//20
	cout << "p2 age: " << p2->getAge() << endl;//20!!static仅初始化一次
	cout << "p1 age: " << p1->getAge() << endl;//20
}
```

或者引用

```cpp
#include <iostream>
using namespace std;

class Name
{
public:
	static Name& creat(int m)
	{
		static Name instance(m);
		return instance;
	}
	int getAge()
	{
		return m_age;
	}
private:
	int m_age;
	Name(int age)
	{
		this->m_age = age;
	}
};

int main()
{
	Name&obj=Name::creat(20);
	Name&obj2=Name::creat(25);
	cout<<"Age of first object is "<<obj.getAge()<<endl;//20
	cout<<"Age of second object is "<<obj2.getAge()<<endl;//20, static仅构造一次
	cout << "Age of first object is " << obj.getAge() << endl;//20
}
```

#### 默认(缺省)构造函数

无参数的、public的
只在用户没有提供自定义构造函数的情况下，才由编译器提供

```cpp
int main()
{
	A  a;//右边任一种形式，均报错
}
```

```cpp
//wrong1
class A
{
public:  
	A(int n){  }
};
```

```cpp
//wrong2
class A
{
private: 
	A(int n){  }
};
```

```cpp
//wrong3
class A
{
public: 
	A();//仅声明, 无具体实现
};
```

##### 解决方案: default(C++11)

该函数比自定义默认构造函数获得更高的代码效率。
=default 函数特性仅适用于类的特殊成员函数，且该特殊成员函数没有默认参数。
=default 函数既可以在类体内定义，也可以在类体外定义

```cpp
#include <iostream>
using namespace std;

class A
{
public: 
	A(int a)
	{

	}
	A()=default;
    /*A(int a)=default;*/  //wrong
};

int main()
{
	A a;
	return 0;
}
```

#### 对象初始化

> [!tip]
>
> 在构造函数内通过赋值初始化
>
> C++11中，类定义时指定初值
>
> 在构造函数的初始化列表中初始化

> [!warning]
>
> 仅当整型(int)或枚举类型(char)的**静态数据**成员被声明为const时, 可以在类定义时指定初值

```cpp
class Card
{
	const float delta = 0.1;
	const int a = 10;

	static const int value = 666;
	static const char suit = 'S';
    static const float rank = 'A';//wrong
    

	static int a = 10;//wrong
	static float sss = 100;//wrong
};
```

##### 初始化列表

> [!warning]
>
> 可以使用初始化列表进行初始化
> 普通数据成员
> **常量数据成员(必须)**
> **引用数据成员(必须)**
> **对象数据成员(没有无参构造函数时，必须)**

#### 非静态数据成员

非静态数据成员
严格按照**定义顺序**依次初始化
先在**初始化列表**中查找是否指定了初始化方式：
若找到，按指定初始化；
若没找到，按无参构造初始化(内置类型的值不确定)
若没找到无参构造函数，则报错（如desc）
**全部完成后**，再进入构造函数的{ }

### 析构函数

访问控制：一般均为public
有this，非static，非const

> [!tip]
>
> 创建对象----访问构造函数
> 销毁对象----访问析构函数
>
> 构造函数的访问
> 显式调用(显式创建对象)
> 隐式调用(自动转换)
> 对象成员的创建
> 析构函数的访问
> 程序区、栈区：生存期结束后，自动执行
> 堆区：需**显式**调用

![image-20240618000757978](C:\Users\10164\AppData\Roaming\Typora\typora-user-images\image-20240618000757978.png)

# chapter 9 拷贝赋值

> [!note]
>
> 拷贝构造函数：
> 从无到有地创建一个新对象
> 不同于普通构造函数，参数必须且只需一个本类对象
> 是一种特殊的构造

```cpp
//wrong: 一种特殊的构造
class Liu
{
public:
	Liu(const Liu& l)
	{
		this->a = l.a;
	}
private:
	int a = 0;
};

int main()
{
	Liu l1;//wrong: 不存在默认构造
}
```

##### 所有权转移的拷贝构造

```cpp
class liu
{
public:
	liu(liu& rhs)
	{
		this->ptr = rhs.ptr;
		rhs.ptr = nullptr;
	}
private:
	liu* ptr = this;
};
```

##### 深拷贝

```cpp
class A
{};

class B
{
public:

	B(const B& b) :pA(new A(*b.pA)), rA(b.rA), aA(b.aA) {}
	
	B(const B& b) : rA(b.rA), aA(b.aA)
	{
		pA = new A(*b.pA);
	}
private:
	A* pA;
	A& rA;
	A aA;
};
```

> [!tip]
>
> #### **需要自定义拷贝构造函数：**
>
> 深拷贝时；
> 禁止拷贝；//自定义一个private且没有实现的拷贝构造函数**或者**My(const My&) = delete;
>
> ![image-20240622231755850](C:\Users\10164\AppData\Roaming\Typora\typora-user-images\image-20240622231755850.png)
>
> 防止按值传递对象；
> 程序员的其它特殊目的(如计数、所有权转移、单件等)

### 默认赋值函数

没有显式给出赋值函数时，由编译器提供
访问控制是public。
采用浅赋值
含义类似浅拷贝
对象成员的赋值
非静态引用或const成员不能赋值

#### 自定义赋值

```cpp
T& operator= ( [const] T & rhs );
```

