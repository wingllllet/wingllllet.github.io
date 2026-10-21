---
title: "C++基础语法"
date: 2025-07-11T17:58:43+08:00
image: https://pub-491d983cd40449fea50cff8b66683e0d.r2.dev/img/{fullName}/20260712125840366.png
tags: ["C++"]
---

## 前言

​	之前为了完成课程作业学过几天时间C++，但现在感觉忘得差不多了，所以重新学习C++基础，不过基于我有基础的情况下应该能用几天速通基础语法。

## 基本语法

### hello world

学语言都是从`Hello World`开始。

```c++
// mian.cpp

#include <iostream>
using namespace std;

int main() {
    cout << "Hello, World!" << endl ;  // 输出  Hello, World!
    return 0;
}

```

```
1. C++语言文件的后缀是 .cpp。
2. C++语言定义的一些头文件，用#include应用，<iostream> 就是一种头文件。
3. using namespace std; 是让编译器使用std命名空间。命名空间则是C++的一个较新的概念。
4.int main() 是主函数，项目文件夹就是从这个函数开始执行的，后面多文件时要注意这个函数。
5. cout << "Hello, World" << endl; 这句会让该字符串显示出来。
6. return 0; 终止main()函数，并调用进程返回值0。
7. ; 分号是语句结束符，只有加上这个符号才表示这句代码写完了。
```

### 标识符

C++的标识符只能由A-Z、a-z、下划线开始，后面跟上零个或多个字母、数字、下划线组成。举几个例子。

```c++
_13 A_dd aaa b12
```

### 空格符

空格符也称空白符，负责分隔语句的各个部分。

### 注释符

注释符会让编译器忽略该行或多行代码。

```
// : 一般用于单行注释
/* ... */ : 用于多行注释
```

```c++
#include <iostream>
using namespace std;

int main() {
	// cout << "whyboy" << endl;		//没有输出
    cout << "Hello, World!" << endl ;  // 输出  Hello, World!
    return 0;
}
```

##  数据类型

| 类型     | 关键字  |
| -------- | ------- |
| 布尔型   | bool    |
| 字符型   | char    |
| 整型     | int     |
| 浮点型   | float   |
| 双浮点型 | double  |
| 无类型   | void    |
| 宽字符型 | wchar_t |

```c++
#include<iostream>  
#include <limits>
 
using namespace std;  
  
int main()  
{  
    cout << "type: \t\t" << "************size**************"<< endl;  
    cout << "bool: \t\t" << "所占字节数：" << sizeof(bool);  
    cout << "\t最大值：" << (numeric_limits<bool>::max)();  
    cout << "\t\t最小值：" << (numeric_limits<bool>::min)() << endl;  
    cout << "char: \t\t" << "所占字节数：" << sizeof(char);  
    cout << "\t最大值：" << (numeric_limits<char>::max)();  
    cout << "\t\t最小值：" << (numeric_limits<char>::min)() << endl;  
    cout << "signed char: \t" << "所占字节数：" << sizeof(signed char);  
    cout << "\t最大值：" << (numeric_limits<signed char>::max)();  
    cout << "\t\t最小值：" << (numeric_limits<signed char>::min)() << endl;  
    cout << "unsigned char: \t" << "所占字节数：" << sizeof(unsigned char);  
    cout << "\t最大值：" << (numeric_limits<unsigned char>::max)();  
    cout << "\t\t最小值：" << (numeric_limits<unsigned char>::min)() << endl;  
    cout << "wchar_t: \t" << "所占字节数：" << sizeof(wchar_t);  
    cout << "\t最大值：" << (numeric_limits<wchar_t>::max)();  
    cout << "\t\t最小值：" << (numeric_limits<wchar_t>::min)() << endl;  
    cout << "short: \t\t" << "所占字节数：" << sizeof(short);  
    cout << "\t最大值：" << (numeric_limits<short>::max)();  
    cout << "\t\t最小值：" << (numeric_limits<short>::min)() << endl;  
    cout << "int: \t\t" << "所占字节数：" << sizeof(int);  
    cout << "\t最大值：" << (numeric_limits<int>::max)();  
    cout << "\t最小值：" << (numeric_limits<int>::min)() << endl;  
    cout << "unsigned: \t" << "所占字节数：" << sizeof(unsigned);  
    cout << "\t最大值：" << (numeric_limits<unsigned>::max)();  
    cout << "\t最小值：" << (numeric_limits<unsigned>::min)() << endl;  
    cout << "long: \t\t" << "所占字节数：" << sizeof(long);  
    cout << "\t最大值：" << (numeric_limits<long>::max)();  
    cout << "\t最小值：" << (numeric_limits<long>::min)() << endl;  
    cout << "unsigned long: \t" << "所占字节数：" << sizeof(unsigned long);  
    cout << "\t最大值：" << (numeric_limits<unsigned long>::max)();  
    cout << "\t最小值：" << (numeric_limits<unsigned long>::min)() << endl;  
    cout << "double: \t" << "所占字节数：" << sizeof(double);  
    cout << "\t最大值：" << (numeric_limits<double>::max)();  
    cout << "\t最小值：" << (numeric_limits<double>::min)() << endl;  
    cout << "long double: \t" << "所占字节数：" << sizeof(long double);  
    cout << "\t最大值：" << (numeric_limits<long double>::max)();  
    cout << "\t最小值：" << (numeric_limits<long double>::min)() << endl;  
    cout << "float: \t\t" << "所占字节数：" << sizeof(float);  
    cout << "\t最大值：" << (numeric_limits<float>::max)();  
    cout << "\t最小值：" << (numeric_limits<float>::min)() << endl;  
    cout << "size_t: \t" << "所占字节数：" << sizeof(size_t);  
    cout << "\t最大值：" << (numeric_limits<size_t>::max)();  
    cout << "\t最小值：" << (numeric_limits<size_t>::min)() << endl;  
    cout << "string: \t" << "所占字节数：" << sizeof(string) << endl;  
    // << "\t最大值：" << (numeric_limits<string>::max)() << "\t最小值：" << (numeric_limits<string>::min)() << endl;  
    cout << "type: \t\t" << "************size**************"<< endl;  
    return 0;  
}
```

## 变量类型

变量就是程序可操作的存储区的名称，常用的变量类型有int、char、float、double、void、bool。

变量声明（extern），它的作用就是声明某个类型的变量，并可以在其他文件中使用该变量。

```c++
#include <iostream>
using namespace std;
 
// 变量声明
extern int a, b;
extern int c;
extern float f;
  
int main ()
{
  // 变量定义
  int a, b;
  int c;
  float f;
 
  // 实际初始化
  a = 10;
  b = 20;
  c = a + b;
 
  cout << c << endl ;
 
  f = 70.0/3.0;
  cout << f << endl ;
 
  return 0;
}
```

## 变量作用域

变量作用域分为全局变量和局部变量。

```c++
#include <iostream>
using namespace std;
 
// 全局变量声明
int g = 20;
 
int main ()
{
  // 局部变量声明
  int g = 10;
 
  cout << g;
 
  return 0;
}

//输出:  10
```

## 常量

常量是固定值，在程序执行期间不会改变。

### 定义常量

```c++
#define identifier value
```

```c++
#include <iostream>
using namespace std;

#define LENGTH 10

int main()
{
	int area;
	
	area = LENGTH * LENGTH;
    cout << "area:" << area << endl;
    cout << "LENGTH:" << LENGTH << endl;
    return 0;
}

// 输出： area:100
//		 LENGTH:10
```

```c++
const type vatiable = value
```

```c++
#include <iostream>
using namespace std;

int main()
{
    const int LENGTH = 10;
    
    int area = LENGTH * LENGTH;
    //错误语句   LENGTH = 9;
    cout << "area:" << area << endl;
    cout << "LENGTH:" << LENGTH << endl;
    return 0;
}
```

## 基本的输入输出

### 标准输入流

```c++
// cin 输入

#include <iostream>
using namespace std;

int main()
{
    char name[9];
    
    cout << "请输入一些字符串：" << endl;

    cin >> name;
    cout << "输入的是：" << name << endl;
    return 0;
}
```

```
标准日志流：  clog
标准错误流：	cerr
```

## 结构体

类似python的字典类型。使用struct语句定义结构体。用`.`来访问里面的成员。

```
struct type_name {
member_type1 member_name1;
member_type2 member_name2;
member_type3 member_name3;
.
.
} object_names;
```

```c++
#include <iostream>
using namespace std;

struct Books
{
    char title[50];
    char author[50];
    int pages;
};

int main()
{
    Books Book1;
    Books Book2;

    strcpy( Book1.title, "winglet");
    strcpy( Book1.author, "winglet"); 
    Book1.pages = 12345;
 
   // Book2 详述
    strcpy( Book2.title, "whyboy");
    strcpy( Book2.author, "whyboy");
    Book2.pages = 12345;
    
    cout << Book1.title <<endl;
    cout << Book1.author <<endl;
    cout << Book1.pages <<endl;
 
   // 输出 Book2 信息
    cout << Book2.title <<endl;
    cout << Book2.author <<endl;
    cout << Book2.pages <<endl;
    return 0;
}

/* 输出：  winglet
winglet
12345
whyboy
whyboy
12345
*/
```

### 结构作为函数参数

```c++
#include <iostream>
#include <cstring>
 
using namespace std;
void printBook( struct Books book );
 
// 声明一个结构体类型 Books 
struct Books
{
   char  title[50];
   char  author[50];
   char  subject[100];
   int   book_id;
};
 
int main( )
{
   Books Book1;        // 定义结构体类型 Books 的变量 Book1
   Books Book2;        // 定义结构体类型 Books 的变量 Book2
 
    // Book1 详述
   strcpy( Book1.title, "C++ 教程");
   strcpy( Book1.author, "Runoob"); 
   strcpy( Book1.subject, "编程语言");
   Book1.book_id = 12345;
 
   // Book2 详述
   strcpy( Book2.title, "CSS 教程");
   strcpy( Book2.author, "Runoob");
   strcpy( Book2.subject, "前端技术");
   Book2.book_id = 12346;
 
   // 输出 Book1 信息
   printBook( Book1 );
 
   // 输出 Book2 信息
   printBook( Book2 );
 
   return 0;
}
void printBook( struct Books book )
{
   cout << "书标题 : " << book.title <<endl;
   cout << "书作者 : " << book.author <<endl;
   cout << "书类目 : " << book.subject <<endl;
   cout << "书 ID : " << book.book_id <<endl;
}
```

## 数据结构

### 数组

```c++
int arr[5] = {1, 2, 3, 4, 5};
cout << arr[0]; // 输出第一个元素
```

### 结构体

```c++
struct Person {
    string name;
    int age;
};
Person p = {"Alice", 25};
cout << p.name << endl; // 输出 Alice
```

### 类(class)

类是面向对象编程的核心结构，允许定义成员变量和成员函数，类似struct。

```c++
class Person {
private:
    string name;
    int age;
public:
    Person(string n, int a) : name(n), age(a) {}
    void printInfo() {
        cout << "Name: " << name << ", Age: " << age << endl;
    }
};
Person p("Bob", 30);
p.printInfo(); // 输出: Name: Bob, Age: 30
```

### 链表

链表是一种动态数据结构，由一系列节点组成，每个节点包含数据和指向下一个节点的指针。

```c++
struct Node {
    int data;
    Node* next;
};
Node* head = nullptr;
Node* newNode = new Node{10, nullptr};
head = newNode; // 插入新节点
```

### 动态数组

```c++
vector<int> v;
v.push_back(1);
v.push_back(2);
cout << v[0]; // 输出 1
```

## 文件和流

| 数据类型 | 描述                                                         |
| -------- | ------------------------------------------------------------ |
| ofstream | 该数据类型表示输出文件流，用于创建文件并向文件写入信息。     |
| ifstream | 该数据类型表示输入文件流，用于从文件读取信息。               |
| fstream  | 该数据类型通常表示文件流，且同时具有 ofstream 和 ifstream 两种功能，这意味着它可以创建文件，向文件写入信息，从文件读取信息。 |

### 打开文件

**open()** 成员函数的第一参数指定要打开的文件的名称和位置，第二个参数定义文件被打开的模式

```c++
void open(const char *filename, ios::openmode mode);
```

| 模式标志  | 功能                                                         |
| --------- | ------------------------------------------------------------ |
| ios:: app | 追加模式。所有的写入都追加到文件末尾                         |
| ios::ate  | 文件打开后定位到文件末尾                                     |
| ios::in   | 打开文件用于读取                                             |
| ios::out  | 打开文件用于写入                                             |
| ios:trunc | 如果该文件已存在，其内容将在打开文件之前被截断，即把文件的长度设为0 |

```c++
//如果您想要以写入模式打开文件，并希望截断文件，以防文件已存在，那么您可以使用下面的语法：
ofstream outfile;
outfile.open("file.dat", ios::out | ios::trunc );
//想要打开一个文件用于读写
ifstream  afile;
afile.open("file.dat", ios::out | ios::in );
```

### 关闭文件

```c++
void close();
```

### 读取和写入

```c++
#include <fstream>
#include <iostream>
using namespace std;
 
int main() {
    
   char data[100];
 
   // 以写模式打开文件
   ofstream outfile;
   outfile.open("afile.dat");
 
   cout << "Writing to the file" << endl;
   cout << "Enter your name: "; 
   cin.getline(data, 100);
 
   // 向文件写入用户输入的数据
   outfile << data << endl;
 
   cout << "Enter your age: "; 
   cin >> data;
   cin.ignore();
   
   // 再次向文件写入用户输入的数据
   outfile << data << endl;
 
   // 关闭打开的文件
   outfile.close();
 
   // 以读模式打开文件
   ifstream infile; 
   infile.open("afile.dat"); 
 
   cout << "Reading from the file" << endl; 
   infile >> data; 
 
   // 在屏幕上写入数据
   cout << data << endl;
   
   // 再次从文件读取数据，并显示它
   infile >> data; 
   cout << data << endl; 
 
   // 关闭打开的文件
   infile.close();
 
   return 0;
}
```

