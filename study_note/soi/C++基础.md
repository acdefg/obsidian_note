down:: [[C++头文件]]
# 基础编程
## 基础结构
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202412191634586.png?token=ALRC6IWHIOB6A6ULZM4AHXLHMPNOA)

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