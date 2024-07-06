6.4 面向对象_task08单例模式

# 单例模式

在C++中，单例模式（Singleton Pattern）是一种创建型设计模式，旨在确保一个**类只有一个实例**，并提供**全局访问点**。这在需要全局状态或资源管理的场景中非常有用，例如日志记录、数据库连接池、配置管理等。

### 单例模式的实现步骤

实现单例模式主要包括以下**三个**步骤：

1. **构造函数私有化**：防止通过外部代码直接创建实例。
2. **删除复制构造函数和赋值运算符**：防止通过复制或赋值创建新的实例。
3. **提供一个访问实例的静态方法**：通常是 `getInstance()` 方法，该方法在第一次调用时创建唯一的实例，并在后续调用中返回该实例。

### 经典的单例实现

下面是一个简单的单例模式实现示例：

```cpp
#include <iostream>
#include <mutex>

class Singleton {
private:
    // 私有构造函数
    Singleton() {
        std::cout << "Singleton instance created." << std::endl;
    }

    // 禁止拷贝构造函数和赋值运算符
    Singleton(const Singleton&) = delete;
    Singleton& operator=(const Singleton&) = delete;

    // 静态指针，指向唯一的实例
    static Singleton* instance;
    static std::mutex mtx;

public:
    // 获取唯一实例的方法
    static Singleton* getInstance() {
        if (instance == nullptr) {
            std::lock_guard<std::mutex> lock(mtx); // 线程安全
            if (instance == nullptr) {
                instance = new Singleton();
            }
        }
        return instance;
    }
};

// 初始化静态成员
Singleton* Singleton::instance = nullptr;
std::mutex Singleton::mtx;

int main() {
    Singleton* s1 = Singleton::getInstance();
    Singleton* s2 = Singleton::getInstance();

    // 输出指针地址，验证两个指针是否指向同一个实例
    std::cout << "s1: " << s1 << std::endl;
    std::cout << "s2: " << s2 << std::endl;

    return 0;
}
```

### 解释

1. **私有构造函数**：防止外部直接创建 `Singleton` 类的实例。
2. **删除拷贝构造函数和赋值运算符**：防止通过复制或赋值创建新的实例。
3. **静态指针 `instance`**：用于指向唯一的 `Singleton` 实例。
4. **静态方法 `getInstance()`**：提供获取唯一实例的途径，使用双重检查锁（Double-checked locking）确保线程安全。

### 线程安全

上述示例中使用了 `std::mutex` 和 `std::lock_guard` 来确保在多线程环境中实例创建的线程安全性。双重检查锁机制（Double-checked locking）通过减少锁的开销来提高性能。

### C++11的懒汉式单例

在C++11及之后的版本中，可以利用静态局部变量的特性简化单例模式的实现，因为静态局部变量在第一次使用时才初始化，并且是线程安全的。

```cpp
class Singleton {
private:
    Singleton() {
        std::cout << "Singleton instance created." << std::endl;
    }

    Singleton(const Singleton&) = delete;
    Singleton& operator=(const Singleton&) = delete;

public:
    static Singleton& getInstance() {
        static Singleton instance;
        return instance;
    }
};

int main() {
    Singleton& s1 = Singleton::getInstance();
    Singleton& s2 = Singleton::getInstance();

    std::cout << "s1: " << &s1 << std::endl;
    std::cout << "s2: " << &s2 << std::endl;

    return 0;
}
```

在这种实现中，静态局部变量 `instance` 保证了线程安全性，并且代码更加简洁。

### [JLUtech]懒汉单例(莫名相似)

```cpp
//狗屁不是, 不想写了
```



# 常对象, 常指针, 常引用

在C++中，常对象（constant object）是指使用 `const` 关键字修饰的对象。一旦对象被声明为常对象，它的状态就不能被修改。也就是说，常对象的成员变量不能被更改，且只能调用其 `const` 成员函数（常函数）。

### 常对象的声明和使用

常对象可以通过在对象声明时添加 `const` 关键字来创建。例如：

```cpp
const MyClass obj(42);
```

在上面的例子中，`obj` 是一个常对象，其状态在生命周期内不能被修改。

### 常对象的限制

1. **不能修改成员变量**：常对象的成员变量不能被修改，即使是通过对象的方法。
2. **只能调用常函数**：常对象只能调用 `const` 成员函数，不能调用非常函数（非 `const` 成员函数）。

### 示例代码

以下是一个包含常对象的示例：

```cpp
#include <iostream>

class MyClass {
private:
    int value;

public:
    // 构造函数
    MyClass(int val) : value(val) {}

    // 常函数
    int getValue() const {
        return value;
    }

    // 非常函数
    void setValue(int val) {
        value = val;
    }
};

int main() {
    const MyClass obj(42);  // 声明常对象

    std::cout << "Value: " << obj.getValue() << std::endl; // OK

    // obj.setValue(84); // 错误：不能调用非常函数

    return 0;
}
```

### 关键点

1. **声明常对象**：使用 `const` 关键字修饰对象声明。
    ```cpp
    const MyClass obj(42);
    ```

2. **调用常函数**：常对象只能调用 `const` 成员函数。
    ```cpp
    std::cout << obj.getValue() << std::endl; // OK
    ```

3. **禁止修改状态**：常对象不能调用非常函数，也不能直接修改成员变量。
    ```cpp
    // obj.setValue(84); // 错误：不能调用非常函数
    ```

### 常指针和常引用

常对象通常与常指针和常引用一起使用，以提供额外的语义约束和安全性。

#### 常引用

常引用指向常对象，保证引用指向的对象状态不会被修改。

```cpp
void printValue(const MyClass& obj) {
    std::cout << obj.getValue() << std::endl;
}

int main() {
    MyClass obj(42);
    printValue(obj); // 传递常引用
    return 0;
}
```

#### 常指针

常指针不能通过指针修改指向的对象。

```cpp
const MyClass* ptr = &obj;
std::cout << ptr->getValue() << std::endl; // OK
// ptr->setValue(84); // 错误：不能调用非常函数
```

### 总结

- **常对象** 是使用 `const` 关键字修饰的对象，其状态在整个生命周期内不能被修改。
- **作用** 是确保对象的不可变性，防止意外修改对象状态，提高代码的安全性和可维护性。
- **限制** 是常对象只能调用 `const` 成员函数，不能调用非常函数或直接修改成员变量。
- **常指针和常引用** 提供了更强的语义约束，使代码更加安全和可靠。

常对象在C++编程中用于保证对象状态的不可变性，是编写稳定和安全代码的重要工具。

# 复用

复用一个已有的类时，应优先选择**组合方式**，而不是继承方式.

组合耦合性更低.

# 异常

异常处理机制是编程语言中用于处理程序运行时错误的一种结构化方式。通过异常处理，程序可以捕获和处理错误，而不是在遇到错误时崩溃。这提高了程序的健壮性和用户体验。

### 异常处理的基本概念

1. **异常（Exception）**：程序运行过程中发生的错误或异常情况，如除零错误、数组越界、文件未找到等。
2. **抛出（Throwing）**：当程序遇到错误时，通过抛出异常通知异常的发生。
3. **捕获（Catching）**：程序中指定的代码块，用于捕获和处理异常。

### C++中的异常处理

在C++中，异常处理通过 `try`、`throw` 和 `catch` 关键字实现。

#### 基本语法

```cpp
try {
    // 可能抛出异常的代码
} catch (const std::exception& e) {
    // 异常处理代码
}
```

#### 示例

```cpp
#include <iostream>
#include <stdexcept>

void divide(int a, int b) {
    if (b == 0) {
        throw std::runtime_error("Division by zero"); // 抛出异常
    }
    std::cout << "Result: " << a / b << std::endl;
}

int main() {
    try {
        divide(10, 2);  // 正常情况
        divide(10, 0);  // 将抛出异常
    } catch (const std::exception& e) {
        std::cerr << "Error: " << e.what() << std::endl;  // 捕获并处理异常
    }
    return 0;
}
```

### 异常处理的步骤

1. **try 块**：包含可能抛出异常的代码。
2. **throw 语句**：在检测到错误时，抛出一个异常对象。
3. **catch 块**：捕获特定类型的异常，并执行相应的处理代码。

### 常见异常类

C++ 标准库中定义了一些常见的异常类，位于 `<stdexcept>` 头文件中，如：

- `std::exception`：所有标准C++异常的基类。
- `std::runtime_error`：表示运行时错误。
- `std::logic_error`：表示逻辑错误。

### 多重捕获

可以为不同类型的异常定义多个 `catch` 块：

```cpp
try {
    // 可能抛出多个不同类型的异常
} catch (const std::runtime_error& e) {
    // 处理 runtime_error 类型的异常
} catch (const std::logic_error& e) {
    // 处理 logic_error 类型的异常
} catch (const std::exception& e) {
    // 处理所有其他 std::exception 派生类的异常
}
```

### 异常处理的优点

1. **代码清晰**：将正常逻辑与错误处理逻辑分离，使代码更清晰易读。
2. **集中处理**：在一个地方处理错误，使代码更易于维护。
3. **提高健壮性**：程序可以优雅地处理错误，而不是在遇到错误时崩溃。

### 异常的名称

常见的异常类包括 `std::invalid_argument`、`std::logic_error`、`std::runtime_error` 以及 `std::exception`。

其他的异常类为前三者的**子类**.

### 尝试整活

```cpp
#include <iostream>
using namespace std;
#include<stdexcept>

void ShalangBaizi()
{
	cout<<"please enter the birthday of Shalang Baizi:"<<endl;
	int n; cin >> n;

	if (n == 430)
		throw runtime_error("Shalang Baizi is a magician!");
	else if (n == 423)
		throw logic_error("Shalang Baizi is a mathematician!");
	else if(n==424)
		throw invalid_argument("Shalang Baizi is a physicist!");
	else if(n==420)
		throw domain_error("Shalang Baizi is a chemist!");
}

int main()
{
	try
	{
		ShalangBaizi();
	}
	catch (runtime_error& e)
	{
		cout << e.what() << endl;
	}
	catch (logic_error& e)
	{
		cout << e.what() << endl;
	}
	catch (invalid_argument& e)
	{
		cout << e.what() << endl;
	}
	catch (domain_error& e)
	{
		cout << e.what() << endl;
	}
	
	return 0;
}
```

### 总结

异常处理机制通过 `try`、`throw` 和 `catch` 关键字提供了一种结构化的错误处理方式。它使得程序可以**捕获和处理运行时错误**，提高了程序的健壮性和可维护性。在C++中，异常处理是一个重要的编程技术，可以帮助程序员编写更健壮和可靠的代码。





# 头文件, 实现文件里面都该写什么, 以及如何正确使用包含警戒

头文件和实现文件在C++编程中扮演了不同的角色，正确使用它们可以提高代码的可读性、可维护性和可重用性。下面我来详细介绍一下它们以及如何正确使用包含警戒。

### 头文件（Header Files）

头文件通常包含了**类声明**、**函数声明**、**常量**、**宏定义**等代码，供其他源文件包含和使用。头文件的主要作用是**声明接口**，使得代码模块化、可重用，并且提供了对外的公共接口。

#### 内容

1. **类声明**：类的成员函数和成员变量的声明。

2. **函数声明**：函数的声明，包括函数名、参数列表和返回类型。

3. **常量声明**：常量的声明，如 `const int MAX_SIZE = 100;`。

4. **宏定义**：宏定义，如 `#define PI 3.1415926`。

5. **内联函数定义**：在头文件中定义的短小的函数可以直接在头文件中实现。

6. [JLUtech]**extern变量声明**: 在头文件加入extern 变量, 并在任意.cpp文件中写入该文件定义

   ```h
   extern int header;
   ```

   ```cpp
   int header = 200;
   ```

#### 示例

```cpp
// MyClass.h

#ifndef MYCLASS_H
#define MYCLASS_H

class MyClass {
public:
    void myFunction(); // 函数声明
    static const int MAX_SIZE = 100; // 常量声明

private:
    int myVariable; // 成员变量
};

#endif
```

### 实现文件（Source Files）

实现文件包含了类的具体实现、函数的具体定义等。它一般用来定义头文件中声明的函数和类的具体实现细节。

#### 内容

1. **类成员函数的定义**：类的**成员函数的具体实现**。
2. **函数定义**：函数的**具体实现**。
3. **全局变量的定义**：全局**变量的定义**，应尽量避免全局变量的使用。
4. **静态成员变量的定义**：静态成员**变量的定义**。

#### 示例

```cpp
// MyClass.cpp

#include "MyClass.h"

void MyClass::myFunction() {
    // 函数的具体实现
}
```

### 包含警戒（Include Guards）

包含警戒是一种预处理指令，用于防止头文件被重复包含。这在大型项目中尤其重要，因为头文件可能会被多个源文件包含，如果不加以控制，可能会导致重复定义的错误。

#### 使用方法

```cpp
#ifndef MYHEADER_H
#define MYHEADER_H

// 头文件内容

#endif
```

### 使用注意事项

1. **只包含必要的内容**：头文件中应该只包含其他文件需要的内容，避免包含过多的内容，导致编译时间增加和不必要的依赖。
2. **避免在头文件中定义函数体**：尽量避免在头文件中定义函数体，否则会导致多个源文件包含同一个头文件时，产生重复定义的错误。
3. **正确使用包含警戒**：在每个头文件的开头和结尾都应该使用包含警戒，以防止头文件被多次包含。

### 总结

- **头文件**用于声明接口，包含类声明、函数声明等。
- **实现文件**用于定义接口的具体实现。
- **包含警戒**用于防止头文件被多次包含。
- **使用时注意**：合理划分头文件和实现文件，只包含必要的内容，并正确使用包含警戒。



# extern用法

`extern` 关键字用于声明一个全局变量或函数，但不定义它。它告诉编译器，该变量或函数*在其他文件中有定义，需要在链接时找到其定义*。`extern` 的使用可以使得变量或函数在多个文件之间共享。

### 使用 `extern` 的场景

1. **声明全局变量**：在一个文件中声明一个全局变量，然后在其他文件中使用 `extern` 声明该变量，以便在链接时找到它的定义。
2. **声明全局函数**：在一个文件中声明一个全局函数，然后在其他文件中使用 `extern` 声明该函数，以便在链接时找到它的定义。

### 示例

假设有两个源文件 `file1.cpp` 和 `file2.cpp`：

```cpp
// file1.cpp
#include <iostream>

// 声明全局变量
extern int globalVariable;

// 声明全局函数
extern void globalFunction();

int main() {
    // 使用全局变量
    std::cout << "Global variable: " << globalVariable << std::endl;
    // 调用全局函数
    globalFunction();
    return 0;
}
```

```cpp
// file2.cpp
#include <iostream>

// 定义全局变量
int globalVariable = 42;

// 定义全局函数
void globalFunction() {
    std::cout << "Global function called" << std::endl;
}
```

### 解释

- 在 `file1.cpp` 中，使用 `extern` 声明了全局变量 `globalVariable` 和全局函数 `globalFunction`。
- 在 `file2.cpp` 中，定义了全局变量 `globalVariable` 和全局函数 `globalFunction`。
- 当编译 `file1.cpp` 和 `file2.cpp` 时，编译器会将 `file1.cpp` 中的 `extern` 声明与 `file2.cpp` 中的定义进行链接。

### 注意事项

1. **仅声明，不定义**：`extern` 声明仅用于声明，不用于定义变量或函数。
2. **一次声明，多次使用**：可以在多个文件中使用同一个 `extern` 声明。
3. **正确链接**：需要确保所有的 `extern` 声明都能够与相应的定义进行正确链接，否则会导致链接错误。

### 总结

`extern` 关键字用于声明全局变量或函数，告诉编译器它们的定义在其他文件中。通过 `extern` 声明，可以实现全局变量和函数在多个文件之间的共享。



# (没啥用但还是写上)

### 虚函数

虚函数是C++中实现多态性的关键概念之一，用于在基类和派生类之间建立**动态绑定的机制**。简单来说，虚函数允许在基类中声明函数，然后在派生类中重新定义该函数，使得通过基类指针或引用调用这个函数时，根据指针或引用实际指向的对象类型来决定调用哪个函数的版本。

在C++中，将基类的函数声明为虚函数的语法如下：

```cpp
class Base {
public:
    virtual void func() {
        // 基类函数实现
    }
};
```

派生类可以覆盖基类中的虚函数，并且使用 `override` 关键字来明确表明这是对基类虚函数的重写：

```cpp
class Derived : public Base {
public:
    void func() override {
        // 派生类函数实现
    }
};
```

通过使用虚函数，可以实现运行时多态性，即在程序运行时根据实际对象的类型来确定调用哪个函数版本，而不是在编译时确定。这种灵活性使得C++能够更好地支持面向对象编程中的多态性特性。



### 虚函数表

假设我们有一个基类 `Shape` 和两个派生类 `Circle` 和 `Rectangle`，它们都包含一个虚函数 `draw()` 用于绘制图形。

```cpp
#include <iostream>

class Shape {
public:
    virtual void draw() {
        std::cout << "Drawing a shape" << std::endl;
    }
};

class Circle : public Shape {
public:
    void draw() override {
        std::cout << "Drawing a circle" << std::endl;
    }
};

class Rectangle : public Shape {
public:
    void draw() override {
        std::cout << "Drawing a rectangle" << std::endl;
    }
};

int main() {
    Shape* shape1 = new Circle();
    Shape* shape2 = new Rectangle();
    
    shape1->draw(); // 输出 "Drawing a circle"
    shape2->draw(); // 输出 "Drawing a rectangle"
    
    delete shape1;
    delete shape2;
    
    return 0;
}
```

在上面的示例中，我们创建了两个指向基类 `Shape` 的指针 `shape1` 和 `shape2`，分别指向派生类 `Circle` 和 `Rectangle` 的对象。当我们调用 `shape1->draw()` 和 `shape2->draw()` 时，由于 `draw()` 是**虚函数**，**程序会根据对象的实际类型调用正确的** `draw()` **函数版本**。

现在让我们来详细阐述虚函数表在这个示例中的作用：

1. **编译阶段**：
   - 当编译器编译 `Circle` 和 `Rectangle` 类时，它**会生成两个虚函数表**，每个表中包含一个指向 `draw()` 函数的指针。
   - 编译器还会在基类 `Shape` 中**生成一个虚函数表**，并在其中填充指向 `Shape::draw()` 函数的指针。
2. **运行时阶段**：
   - 当我们创建 `Circle` 或 `Rectangle` 的对象时，对象的内存布局中会有一个指向虚函数表的**指针**（**虚指针**）。
   - 当我们调用 `shape1->draw()` 时，程序会使用 `shape1` 对象的**虚指针**来**确定正确的虚函数表**，并从中找到指向 `Circle::draw()` 函数的指针，并调用该函数。
   - 同样，当我们调用 `shape2->draw()` 时，程序会使用 `shape2` 对象的虚指针来确定正确的虚函数表，并从中找到指向 `Rectangle::draw()` 函数的指针，并调用该函数。

这样，通过虚函数表的机制，我们实现了基类指针在运行时根据实际对象的类型来调用正确的函数版本，实现了多态性的特性。



### 基类虚函数表的作用

在基类 `Shape` 中生成一个虚函数表，并在其中填充指向 `Shape::draw()` 函数的指针的作用在于为**基类和其派生类**提供一个**统一的接口**。具体来说，这个虚函数表的作用有以下几点：

1. **定义接口**：虚函数表定义了基类 `Shape` 的接口，即它表示了所有派生类都应该具有的公共行为。在虚函数表中，通过将 `draw()` 函数的指针指向 `Shape::draw()` 函数，表明了所有派生类都应该实现 `draw()` 函数，并且应该具有相同的接口。

2. **实现多态**：虚函数表为运行时多态性提供了基础。当我们使用**基类指针或引用调用虚函数时，实际调用的是对象的虚函数表中相应函数的*版本***。在基类中定义虚函数表，并将其填充为指向基类函数的指针，保证了即使通过基类指针或引用调用虚函数时，也能够调用到正确的函数版本。

3. **提供默认实现**：虚函数表中指向 `Shape::draw()` 函数的指针，提供了一个默认的实现。如果某个派生类没有覆盖 `draw()` 函数，那么将会调用基类的 `draw()` 函数，保证了程序的正确性和稳定性。

总的来说，基类 `Shape` 中生成一个虚函数表，并在其中填充指向 `Shape::draw()` 函数的指针，使得基类和其派生类之间建立了一个统一的接口，并为实现多态性提供了基础。



# 文件1：《面向对象程序设计》_11_动态内存管理.pptx

#### 内容概述
该PPT主要介绍了动态内存管理在面向对象程序设计中的应用，具体内容包括内存分配与释放、内存泄漏的预防、指针与引用的使用等。

#### 主要内容
1. **动态内存管理的基本概念**：
   - 动态内存分配是指在程序运行时根据需要分配和释放内存。
   - C++中主要使用`new`和`delete`操作符进行动态内存管理。

2. **内存分配与释放**：
   - 使用`new`操作符为单个变量或数组分配内存。
   - 使用`delete`操作符释放为单个变量或数组分配的内存。
   - 示例代码展示了如何正确使用`new`和`delete`进行内存管理。

3. **内存泄漏**：
   - 内存泄漏指程序在不再需要某块动态分配的内存时未能及时释放，导致内存无法被回收利用。
   - 介绍了预防内存泄漏的方法，例如：使用智能指针（如`std::unique_ptr`和`std::shared_ptr`）管理动态内存。

4. **指针与引用**：
   - 讲解了指针与引用的基本概念及其区别。
   - 介绍了如何使用指针和引用进行动态内存管理，并提供了示例代码。

5. **智能指针**：
   - 详细介绍了C++11引入的智能指针，包括`std::unique_ptr`、`std::shared_ptr`和`std::weak_ptr`。
   - 讨论了智能指针的使用场景及其优缺点。

6. **动态内存管理的常见问题**：
   - 讨论了常见的动态内存管理问题，如野指针、重复释放内存等，并提供了相应的解决方案。

# 文件2：《面向对象程序设计》_14_类的设计.pptx

#### 内容概述
该PPT主要介绍了面向对象编程中类的设计原则与方法，重点讨论了抽象与封装、信息隐蔽、类的使用与实现分离、以及单个类和多个类的设计等内容。

#### 主要内容
1. **面向对象的三大基本特征**：
   - **封装与信息隐蔽**：通过类和对象对客观事物进行抽象，隐藏内部实现细节，只公开必要的行为和属性。
   - **继承**：子类继承父类的属性和方法，支持代码重用。
   - **多态**：通过接口实现不同子类的多态性行为。
2. **抽象与表示**：
   - 讲解了类型、行为和数据的抽象及其在类设计中的应用。
   - 提供了小球碰撞问题和游戏中的怪物战斗问题作为例子，展示了如何将抽象的概念具体化为类的实现。
3. **信息隐蔽**：
   - 讨论了如何通过隐藏类的内部实现细节，提高类的安全性和可维护性。
   - 提供了示例代码，展示了如何隐藏类的私有成员和方法。
4. **分离使用和实现**：
   - 强调了类的使用者与实现者之间的分离，减少耦合度。
   - 提供了示例代码，展示了如何将类的声明和实现分离开来。
5. **多个类的设计**：
   - 讨论了类之间的关系，包括类的拆分与合并、数据关系和行为关系。
   - 提供了一个应用程序的例子，展示了如何设计多个类并处理它们之间的关系。
6. **单个类的设计**：
   - 讨论了单个类的设计原则，包括行为的可见性、行为参数的抽象与封装、依赖与关联、实现的抽象与封装、数据的抽象与封装、类方法与实例方法等。
   - 提供了示例代码，展示了如何设计一个功能完善的类。
7. **创建方法与构造函数**：
   - 介绍了创建方法和构造函数的设计原则。
   - 讨论了如何使用工厂方法模式创建对象，提供了示例代码。
8. **单件模式的实现**：
   - 讲解了单件模式的概念及其实现方法，确保类只有一个实例。
   - 提供了使用静态成员变量和方法实现单件模式的示例代码。

