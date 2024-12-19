
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
  
类的成员函数是指那些把定义和原型写在类定义内部的函数，就像类定义中的其他变量一样。

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