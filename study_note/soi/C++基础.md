down:: [[C++头文件]]
down:: [[刷题记录]]

# 基础编程
## 基础结构
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202412191634586.png?token=ALRC6IWHIOB6A6ULZM4AHXLHMPNOA)

```cpp
//计算数组长度
int len = sizeof(arr) / sizeof(int);
```

## 数据类型
1 byte = 8 bits
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202412191702545.png?token=ALRC6IUC2EZ3EZH22ZEC7NTHMPQUU)
一些基本类型可以使用一个或多个类型修饰符进行修饰：
-   signed
-   unsigned
-   short
-   long
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202412191702522.png?token=ALRC6IXNSO5LA6H3MT5R62DHMPQXG)
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202412191703722.png?token=ALRC6IVOVRPSRBEWYC4KRB3HMPQYI)
int 以及 unsigned int 范围为十进制 10 位数，2\*e10 4\*e10    4 个字节 32bit
short int 范围为十进制 5 位数 3\*e5 2 个字节 16bit
long int  范围为十进制 19 位数 9\*e19 8 个字节 64bit


## typdef 和枚举
### typedef 声明

您可以使用 **typedef** 为一个已有的类型取一个新的名字。下面是使用 typedef 定义一个新类型的语法：

`typedef type newname;` 

例如，下面的语句会告诉编译器，feet 是 int 的另一个名称：

`typedef int feet;`

现在，下面的声明是完全合法的，它创建了一个整型变量 distance：

`feet distance;`

### 枚举类型

枚举类型声明一个可选的类型名称和一组标识符，用来作为该类型的值。其带有零个或多个标识符可以被用来作为该类型的值。每个枚举数是一个枚举类型的常数。

创建枚举，需要使用关键字 **enum**。枚举类型的一般形式为：

`enum enum-name { list of names } var-list;` 

在这里，enum-name 是枚举类型的名称。名称列表 { list of names } 是用逗号分隔的。

例如，下面的代码定义了一个颜色枚举，变量 c 的类型为 color。最后，c 被赋值为 "blue"。

`enum color { red, green, blue } c; c = blue;`

默认情况下，第一个名称的值为 0，第二个名称的值为 1，第三个名称的值为 2，以此类推。但是，您也可以给名称赋予一个特殊的值，只需要添加一个初始值即可。例如，在下面的枚举中，**green** 的值为 5。

`enum color { red, green=5, blue };`

在这里，**blue** 的值为 6，因为默认情况下，每个名称都会比它前面一个名称大 1。



## 输出格式😱

```
#include <iomanip>
//输出浮点数 `cost`，并格式化成固定小数点形式，保留 1 位小数
cout << setiosflags(ios::fixed) << setprecision(1) << cost << endl;
```

```cpp
`printf(``"%.1f %.1f"``,sum,h);`
```
#### example1
```cpp
#include <iostream>

#include <iomanip>

using namespace std;

  

int main() {

    double price;

    cin >> price;

  

    double cost = 0.0;

  

    // write your code here.......

    if(price >= 5000)

        cost = price * 0.6;

    else if(price >= 2000)

        cost = price * 0.7;

    else if(price >= 500)

        cost = price * 0.8;

    else if(price >= 100)

        cost = price * 0.9;

    else

        cost = price;

  

    cout << setiosflags(ios::fixed) << setprecision(1) << cost << endl;

  

    return 0;

}
```


## 函数传参⭐
在 C++ 中，函数传参的方式主要包括以下四种：按值传递（Pass by Value）、按引用传递（Pass by Reference）、按指针传递（Pass by Pointer）、以及按常引用传递（Pass by Const Reference）

### 形参
值传递，swap1函数使用的是`值传递`，它接受两个整数参数x和y，并在函数内部进行交换。这种情况下，函数内部交换的是局部变量的值，而不会影响到main函数中的first和second。
```cpp
void swap1(int x, int y){
    int temp = x;
    x = y;
    y = temp;
}
```

-   **优点**：避免修改原始数据，安全。
-   **缺点**：传递大型对象时效率低，因为需要复制数据。

### 引用传参
传递参数的引用，函数操作的就是原始变量本身，修改会直接影响原变量。
```cpp
#include <iostream>
using namespace std;

void increment(int& x) {
    x += 1;  // 直接操作原始变量
}

int main() {
    int a = 5;
    increment(a); // a 的值会被修改
    cout << "Value of a: " << a << endl; // 输出 6
    return 0;
}

```
-   **优点**：避免拷贝，提高效率；可以直接修改原始数据。
-   **缺点**：可能无意间修改了原始数据。

### 指针传参
将变量的地址传递给函数，函数通过解引用操作访问原始数据。
```cpp
#include <iostream>
using namespace std;

void increment(int* x) {
    if (x) { // 检查指针是否为空
        *x += 1;  // 修改指针指向的变量
    }
}

int main() {
    int a = 5;
    increment(&a); // 传递变量的地址
    cout << "Value of a: " << a << endl; // 输出 6
    return 0;
}
```
-   **优点**：可以传递动态分配的内存或大型结构；显式地指明函数可能修改数据。
-   **缺点**：需要处理指针的合法性（如空指针）。

### 按常引用传递
以引用的方式传递，但不能修改原始数据。
```cpp
#include <iostream>
using namespace std;

void print(const int& x) {
    cout << "Value: " << x << endl;
    // x += 1; // 会报错，因为 x 是 const 引用
}

int main() {
    int a = 5;
    print(a); // 只读访问 a
    return 0;
}
```
-   **优点**：避免拷贝，提高效率；适用于只读操作，安全性更高。
-   **缺点**：不能修改原始数据。

### 总结
-   **按值传递**适用于简单数据类型且不需要修改原始变量的情况。
-   **按引用传递**适用于需要修改原始数据的场景，或避免拷贝大对象。
-   **按指针传递**用于需要操作动态分配的内存，或当可能传递空值时。
-   **按常引用传递**适合只读访问的大型对象，避免拷贝，提高效率。
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202412201445675.png?token=ALRC6IUA4F24NYFVCUXZ2EDHMUJNC)


## 结构体
### 结构体定义和使用
```cpp
struct [structure tag]{
    member definition;
    member definition;
    ...
    member definition;
}[one or more structure variables];  
```

**structure tag** 是可选的，每个 member definition 是标准的变量定义，比如 int i; 或者 float f; 或者其他有效的变量定义。在结构定义的末尾，最后一个分号之前，您可以指定一个或多个结构变量，这是可选的。下面是声明 Book 结构的方式：
```cpp
struct Books{
    char title[50];
    char author[50];
    char subject[100];
    int book_id;
}book; 
```

为了访问结构的成员，我们使用**成员访问运算符（.）**
```cpp
strcpy( Book1.title, "Learn C++ Programming"); 
strcpy( Book1.author, "Chand Miyan"); 
strcpy( Book1.subject, "C++ Programming"); 
Book1.book_id = 6495407;
```

### 指向结构的指针
您可以定义指向结构的指针，方式与定义指向其他类型变量的指针相似，如下所示：
`struct Books *struct_pointer;`

现在，您可以在上述定义的指针变量中存储结构变量的地址。为了查找结构变量的地址，请把 & 运算符放在结构名称的前面，如下所示：
`struct_pointer = &Book1;`

为了使用指向该结构的指针访问结构的成员，您必须使用 -> 运算符，如下所示：
`struct_pointer->title;`

### typedef 关键字
下面是一种更简单的定义结构的方式，您可以为创建的类型取一个"别名"。例如：
```cpp
typedef struct
{
   char  title[50];
   char  author[50];
   char  subject[100];
   int   book_id;
}Books;
```
现在，您可以直接使用 _Books_ 来定义 _Books_ 类型的变量，而不需要使用 struct 关键字。下面是实例：
`Books Book1, Book2;`
您可以使用 **typedef** 关键字来定义非结构类型，如下所示：
`typedef long int *pint32;   pint32 x, y, z;`
x, y 和 z 都是指向长整型 long int 的指针。

## 字符串

```
int sublen = strlen(substr);   //计算char数组到'\0'前实际长度，也就是不计算'\0'在内
```


# 简单概念解释
## C++ 类 & 对象
### 类
定义一个类需要使用关键字 class，然后指定类的名称，并类的主体是包含在一对花括号中，主体包含类的成员变量和成员函数。

定义一个类，本质上是定义一个数据类型的蓝图，它定义了类的对象包括了什么，以及可以在这个对象上执行哪些操作
[C++ 类 & 对象 | 菜鸟教程](https://www.runoob.com/cplusplus/cpp-classes-objects.html)
### 对象
类提供了对象的蓝图，所以基本上，对象是根据类来创建的。声明类的对象，就像声明基本类型的变量一样。下面的语句声明了类 Box 的两个对象：

```cpp
Box Box1; // 声明 Box1，类型为 Box 
Box Box2; // 声明 Box2，类型为 Box
```
对象 Box1 和 Box2 都有它们各自的数据成员。

#### 访问数据成员

类的对象的公共数据成员可以使用直接成员访问运算符 . 来访问。

![](https://www.runoob.com/wp-content/uploads/2015/05/cpp-classes-objects-2020-12-10-11-2.png)


### public & private
public 可以在类外被访问
private 只可以在类的内部被访问

### 成员函数
  
类的成员函数是指 那些把定义和原型写在类定义内部的函数，就像类定义中的其他变量一样。

### example1
```cpp
#include <iostream>
 
using namespace std;
 
class Box
{
   public:
      double length;   // 长度
      double breadth;  // 宽度
      double height;   // 高度
      // 成员函数声明
      double get(void);
      void set( double len, double bre, double hei );
};
// 成员函数定义
double Box::get(void)
{
    return length * breadth * height;
}
 
void Box::set( double len, double bre, double hei)
{
    length = len;
    breadth = bre;
    height = hei;
}
int main( )
{
   Box Box1;        // 声明 Box1，类型为 Box
   Box Box2;        // 声明 Box2，类型为 Box
   Box Box3;        // 声明 Box3，类型为 Box
   double volume = 0.0;     // 用于存储体积
 
   // box 1 详述
   Box1.height = 5.0; 
   Box1.length = 6.0; 
   Box1.breadth = 7.0;
 
   // box 2 详述
   Box2.height = 10.0;
   Box2.length = 12.0;
   Box2.breadth = 13.0;
 
   // box 1 的体积
   volume = Box1.height * Box1.length * Box1.breadth;
   cout << "Box1 的体积：" << volume <<endl;
 
   // box 2 的体积
   volume = Box2.height * Box2.length * Box2.breadth;
   cout << "Box2 的体积：" << volume <<endl;
 
 
   // box 3 详述
   Box3.set(16.0, 8.0, 12.0); 
   volume = Box3.get(); 
   cout << "Box3 的体积：" << volume <<endl;
   return 0;
}
```
当上面的代码被编译和执行时，它会产生下列结果：
```
Box1 的体积：210
Box2 的体积：1560
Box3 的体积：1536
```
需要注意的是，私有的成员和受保护的成员不能使用直接成员访问运算符 (.) 来直接访问。我们将在后续的教程中学习如何访问私有成员和受保护的成员。