
# 头文件类型
在一个正常的 C++ 程序中，需要包含的头文件取决于你程序的功能需求。C++ 标准库提供了丰富的功能，可以帮助你完成各种操作。常见的头文件包括输入输出、容器、算法、数学函数等。下面是一些常见的 C++ 头文件及其用途，帮助你了解在不同情况下应包含哪些头文件。

## 常见文件类型

### 1. **基本输入输出**

-   **`<iostream>`**：提供基本的输入输出功能，支持 `std::cin`、`std::cout`、`std::cerr` 等。
    
 ```cpp
   `#include <iostream>  // 用于输入输出`
```

	 
```cpp
    `std::cout << "Hello, World!" << std::endl; int x; std::cin >> x;`
```
    
-   **`<iomanip>`**：用于格式化输出，比如设置输出宽度、精度等。
    
    cpp
    
    复制代码
    
```cpp
    `#include <iomanip> std::cout << std::fixed << std::setprecision(2) << 3.14159;`
```
    
-   **`<fstream>`**：用于文件操作，如读取和写入文件。
    
    cpp
    
    复制代码
    
```cpp
    `#include <fstream>  // 文件输入输出 std::ofstream outFile("output.txt"); outFile << "Hello, File!" << std::endl;`
```
    

### 2. **容器**

C++ 标准库提供了多种容器（如数组、链表、栈、队列、映射等），你可以根据需求包含不同的头文件。

-   **`<vector>`**：提供 `std::vector` 类，用于存储动态大小的数组。
    
    cpp
    
    复制代码
    
```cpp
    `#include <vector>  // 动态数组 std::vector<int> v = {1, 2, 3}; v.push_back(4);`
```
    
-   **`<list>`**：提供 `std::list` 类，用于双向链表。
    
    cpp
    
    复制代码
    
```cpp
    `#include <list>  // 双向链表 std::list<int> l = {1, 2, 3}; l.push_back(4);`
```
    
-   **`<deque>`**：提供 `std::deque` 类，用于双端队列。
    
    cpp
    
    复制代码
    
```cpp
    `#include <deque>  // 双端队列 std::deque<int> d = {1, 2, 3}; d.push_front(0);`
```
    
-   **`<stack>`**：提供 `std::stack` 类，栈容器。
    
    cpp
    
    复制代码
    
```cpp
  #include <stack>  // 栈 
  std::stack<int> s; 
  s.push(1); s.push(2);`
```
    
-   **`<queue>`**：提供 `std::queue` 类，队列容器。
    
```cpp
#include <queue>  // 队列 
std::queue<int> q;
q.push(1); q.push(2);
```
    
-   **`<map>`** 和 **`<unordered_map>`**：提供关联容器 `std::map` 和 `std::unordered_map`，用来存储键值对。

```cpp
 #include <map>  // 有序map
 std::map<int, std::string> m; 
 m[1] = "One"; m[2] = "Two";
```
    
    
```cpp
#include <unordered_map>  // 无序map 
std::unordered_map<int, std::string> um; 
um[1] = "One"; 
um[2] = "Two";
```
    
-   **`<set>`** 和 **`<unordered_set>`**：分别提供有序集合和无序集合。
    
```cpp
#include <set>  // 有序集合 
std::set<int> s = {1, 2, 3};
```
    

### 3. **算法**

-   **`<algorithm>`**：提供许多常用的算法，如排序、查找、比较等。

```cpp
#include <algorithm>  // 排序、查找等算法 
std::sort(v.begin(), v.end());
```
    
-   **`<functional>`**：提供函数对象、函数适配器等工具，支持 `std::bind`、`std::function` 等。
    
```
#include <functional> 
std::function<int(int, int)> add = [](int a, int b) { return a + b; };
```
    

### 4. **数学函数**

-   **`<cmath>`**：提供常用数学函数，如三角函数、对数、幂运算等。
    
```
#include <cmath>  // 数学函数 
double result = std::sqrt(16.0);  // 计算平方根
```
    
-   **`<cstdlib>`**：提供一些通用的工具函数，如内存分配、随机数生成、转换等。

```
#include <cstdlib>  // 随机数、转换等 
int x = std::rand();
```
    
-   **`<ctime>`**：用于时间和日期相关的函数，如获取当前时间、设置定时器等。
    
```
#include <ctime>  // 时间处理 
std::time_t t = std::time(0);   // 当前时间戳
```
    
### 5. **字符串处理**

-   **`<string>`**：提供 `std::string` 类，用于处理字符串。
    
```
#include <string>  // 字符串类 
std::string str = "Hello, World!";
```
    
-   **`<cstring>`**：提供 C 风格字符串处理函数，如 `strlen`、`strcpy` 等。
    
```
#include <cstring>  // C 风格字符串处理 
char str[20] = "Hello"; 
std::cout << std::strlen(str) << std::endl;
```

### 6. **输入输出流**

-   **`<sstream>`**：提供字符串流，可以将字符串转换为其他数据类型，或者反向操作。
    
```
#include <sstream>  // 字符串流 
std::stringstream ss; 
ss << 42; 
int num; 
ss >> num;
```
    

### 7. **智能指针和内存管理**

-   **`<memory>`**：提供智能指针（`std::shared_ptr`、`std::unique_ptr`）以及内存管理功能。
```
#include <memory>  // 智能指针 
std::shared_ptr<int> ptr = std::make_shared<int>(10);
```
    

### 8. **异常处理**

-   **`<stdexcept>`**：提供标准异常类，用于异常处理。
    
```
#include <stdexcept>  // 异常类 
throw std::out_of_range("Out of range error");
```
    
### 9. **线程和并发编程**

-   **`<thread>`**：提供多线程支持。
    
```
#include <thread>  // 线程操作 
std::thread t([]() { std::cout << "Thread running" << std::endl; }); t.join();
```
    
-   **`<mutex>`**：提供互斥量和锁，用于线程同步。
    
```
#include <mutex>  // 互斥锁 
std::mutex mtx; mtx.lock(); 
// 临界区代码 
mtx.unlock();
```

### 总结

在一个 C++ 程序中，包含哪些头文件取决于你所需的功能。最常用的头文件包括：

-   `<iostream>`：输入输出流。
-   `<vector>`、`<list>`、`<deque>`、`<map>` 等：各种容器。
-   `<algorithm>`：常用算法。
-   `<cmath>`、`<cstdlib>`：数学和实用函数。
-   `<string>`、`<cstring>`：字符串处理。
-   `<memory>`：智能指针。

## stdio
`<stdio.h>` 是 C 语言中一个非常重要的标准头文件，代表 **"Standard Input/Output"**，即标准输入输出库。它定义了一系列用于处理输入输出操作的函数和宏。

在 C++ 中，虽然使用 `<iostream>` 更常见，但 `<stdio.h>` 仍然可以使用，并且在某些低层次程序中可能是更高效的选择。

---

### `<stdio.h>` 的主要内容

#### 1. **输入/输出函数**

-   **标准输入输出：**
    -   `printf()`：格式化输出到标准输出（通常是屏幕）。
    -   `scanf()`：从标准输入（通常是键盘）中读取格式化数据。
-   **文件操作：**
    -   `fopen()`：打开文件。
    -   `fclose()`：关闭文件。
    -   `fread()`/`fwrite()`：从文件中读取或向文件写入数据。
    -   `fgets()`/`fputs()`：从文件读取一行字符串或将字符串写入文件。
-   **低级字符操作：**
    -   `getc()`/`putc()`：从文件中读取或写入一个字符。
    -   `getchar()`/`putchar()`：从标准输入读取或写入一个字符。

#### 2. **缓冲和控制函数**

-   `fflush()`：清空文件缓冲区。
-   `setbuf()`/`setvbuf()`：设置文件流缓冲区。

#### 3. **文件状态和错误处理**

-   `fseek()`：在文件中移动文件指针。
-   `ftell()`：获取文件指针当前位置。
-   `feof()`：检测文件是否到达末尾。
-   `ferror()`：检测文件是否发生错误。

#### 4. **预定义流**

-   `stdin`：标准输入流。
-   `stdout`：标准输出流。
-   `stderr`：标准错误输出流。

---

### 示例代码

#### 1. 使用标准输入输出

```cpp
#include <stdio.h>  
int main() {     
	int num;    
	printf("Enter a number: ");     
	scanf("%d", &num); // 从用户输入中读取整数     
	printf("You entered: %d\n", num); // 打印用户输入的数字     
	return 0; 
}
```

#### 2. 文件操作
```cpp
#include <stdio.h>  
int main() {     
	FILE *file = fopen("example.txt", "w"); // 打开文件进行写操作     
	if (file == NULL) {         
		perror("Error opening file");         
		return -1;     
	}     
	fprintf(file, "Hello, World!\n"); // 写入到文件     
	fclose(file); // 关闭文件     
	return 0; 
}
```

#### 3. 读取文件内容

```cpp
#include <stdio.h>  
int main() {     
	FILE *file = fopen("example.txt", "r"); // 打开文件进行读操作     
	if (file == NULL) {         
		perror("Error opening file");         
		return -1;     
	}     
	char buffer[256];     
	while (fgets(buffer, sizeof(buffer), file)) { 
		// 逐行读取文件内容         
		printf("%s", buffer); 
		// 打印到屏幕     
	}     
	fclose(file); // 关闭文件     
	return 0; 
}
```

---

### 总结

`<stdio.h>` 提供了最基础的输入输出功能，适用于标准输入输出和文件操作。在 C++ 中，如果需要更高级和面向对象的方式，可以使用 `<iostream>`。但对于一些底层程序，`<stdio.h>` 的函数依然是不可替代的。