---
tags: [cpp]
---

# C++

## `std::string` 怎么用？ #cpp #stl

> **速查**
> - `string` 本质是**字节序列**，不是字符序列：`size()` 是字节数，一个汉字在 UTF-8 里占 3 字节
> - 读一整行用 `getline(cin, s)`，读一个词用 `cin >> s`（遇空白就停）
> - 拼接用 `+=` / `append`；循环里写 `s = s + c` 是 O(n²)
> - `find` 失败返回 `string::npos`，不是 -1；判断写 `s.find(x) != string::npos`
> - `substr(pos, len)` 的 `pos` 越界会抛异常，`len` 超长则自动截断

### 正文

**① 声明与初始化**

```cpp
string s;                      // 空串
string a = "hello";            // 从字面量
string b(5, 'a');              // "aaaaa"，5 个字符
string c(a, 1, 3);             // "ell"，从下标 1 起取 3 个
string d = a.substr(1, 3);     // 同上
string e = a;                  // 拷贝

char ch  = a[0];               // 不检查越界
char ch2 = a.at(0);            // 越界抛 std::out_of_range
```

注意 `string s = 'a';` 编译不过——`char` 不是 `string`。要写 `string(1, 'a')` 或 `"a"`。

**② 输入：`>>` 与 `getline` 的分工**

```cpp
cin >> s;              // 跳过前导空白，读到下一个空白为止
getline(cin, s);       // 读一整行（含空格），丢弃行尾换行
```

混用会踩坑：

```cpp
int n;
cin >> n;              // 读到 n 后，换行符还留在缓冲区
getline(cin, s);       // 立刻读到这个换行 → s 是空串
```

两种修法：

```cpp
cin.ignore(numeric_limits<streamsize>::max(), '\n');   // 丢掉到行尾
// 或者
cin >> ws;                                             // 吃掉前导空白（含换行）
getline(cin, s);
```

**③ 常用操作**

| 目的 | 写法 | 备注 |
|---|---|---|
| 长度 | `s.size()` / `s.length()` | 返回 `size_t`（无符号） |
| 判空 | `s.empty()` | 语义比比长度更清楚 |
| 拼接 | `s += t` / `s.append(t)` | 均摊线性 |
| 比较 | 直接用比较运算符 | 按字典序逐字节比较 |
| 查找 | `s.find(t)` / `s.rfind(t)` | 失败返回 `string::npos` |
| 取子串 | `s.substr(pos, len)` | `pos` 越界抛异常 |
| 替换 | `s.replace(pos, len, t)` | 长度可不等 |
| 插入 | `s.insert(pos, t)` | 插在 `pos` 之前 |
| 删除 | `s.erase(pos, len)` | 只给迭代器则删到末尾 |
| 清空 | `s.clear()` | |
| 改长度 | `s.resize(n, ch)` | 变长补 `ch` |
| 首尾 | `s.front()` / `s.back()` | |
| 转数字 | `stoi` / `stol` / `stoll` / `stod` | 非法输入抛 `invalid_argument` |
| 转字符串 | `to_string(x)` | 浮点固定 6 位小数 |
| 转 C 字符串 | `s.c_str()` | 返回 `const char*`，下次修改后失效 |

`stoi` 的一个细节：它**从第一个字符开始解析，遇到非数字就停**，所以 `stoi("12abc")` 返回 `12` 而不抛异常；只有开头就不是数字（如 `"abc"`）才抛 `invalid_argument`，数值超出 `int` 范围才抛 `out_of_range`。

**④ 遍历与刷题常用**

```cpp
for (char c : s) use(c);                 // 只读
for (char& c : s) c = toupper(c);        // 可改
for (int i = 0; i < (int)s.size(); ++i)  // 下标遍历，注意强转
    use(s[i]);

reverse(s.begin(), s.end());             // 反转
sort(s.begin(), s.end());                // 按字符排序

s.erase(unique(s.begin(), s.end()), s.end());   // 先排序才能去重
```

按分隔符切分：

```cpp
stringstream ss(line);
string tok;
while (getline(ss, tok, ',')) { /* tok 是一段 */ }
```

或者按空白切：

```cpp
stringstream ss(line);
string tok;
while (ss >> tok) { /* 自动跳过空白 */ }
```

查找所有出现位置：

```cpp
for (size_t p = s.find(t); p != string::npos; p = s.find(t, p + t.size()))
    cout << p << '\n';
```

**⑤ 七个坑**

1）`find` 的结果是无符号的

```cpp
if (s.find("x") != string::npos) {}   // ✓
if (s.find("x") != -1) {}             // ✗ 有符号/无符号比较，行为不直观
```

2）`s.size() - 1` 在空串时下溢成巨大值

```cpp
for (int i = (int)s.size() - 1; i >= 0; --i)   // 先转 int 再减
```

3）循环里反复 `s = s + c` 是 O(n²)。写 `s += c`，或用 `s.reserve(n)` 预留。

4）`getline` 与 `>>` 混用（见 ②）。

5）`substr` 的 `pos` 可以等于 `size()`（得到空串），但大于 `size()` 会抛 `out_of_range`。

6）中文按字节处理：`s.size()` 是字节数，`s[i]` 是字节不是字符。要按字符处理得用 `wstring` 或专门的 UTF-8 库。

7）`to_string` 控制不了浮点精度：`to_string(3.14)` 得到 `"3.140000"`。要格式化用 `ostringstream`：

```cpp
ostringstream os;
os << fixed << setprecision(2) << 3.14159;   // "3.14"
string out = os.str();
```

**⑥ 性能：SSO 与扩容**

- **SSO（小字符串优化）**：短串直接存在对象内部的缓冲区里，不分配堆内存。libstdc++ 和 MSVC 的阈值是 15 字节，libc++ 是 22 字节。因此 `string` 对象本身通常有 24–32 字节。
- **扩容**：`append` / `+=` 超出容量时按倍数（libstdc++ 约 2 倍）重新分配并搬迁，所以均摊 O(1)。已知最终长度就 `reserve(n)`，省掉多次搬迁。
- **`c_str()` 的指针**在后续任何修改（包括 `+=`）之后都可能失效，不要长期持有。

### 自测

- [ ] `cin >> n` 之后直接 `getline(cin, s)` 为什么会拿到空串？两种修法分别是什么？
- [ ] `s.find(x) != -1` 为什么是错的？应该写成什么？
- [ ] 空串上执行 `s.size() - 1` 得到什么？为什么倒序遍历要先转 `int`？
- [ ] 为什么循环里 `s = s + c` 比 `s += c` 慢一个量级？
