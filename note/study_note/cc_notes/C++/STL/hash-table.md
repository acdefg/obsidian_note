---
tags: [cpp]
created: 星期六, 十月 24日 2026, 11:14:12
modified: 星期六, 十月 24日 2026, 11:15:12
---

# C++

## 哈希表有几种写法？ #cpp #stl

> **速查**
> - STL 里的哈希表只有 `unordered_map` / `unordered_set`；`map` / `set` 是红黑树，不是哈希
> - `unordered_map` 底层是**桶数组 + 链表节点**（链地址法）；标准要求 rehash 后元素引用不失效
> - 更快的工程替代：`absl::flat_hash_map`、`boost::unordered_flat_map`——开放寻址，缓存友好
> - 竞赛写法：PBDS `gp_hash_table`（线性探测）/ 手写 `head[] + nxt[]` 数组拉链
> - 平均 O(1) 的前提是哈希均匀；最坏 O(n)，可被恶意构造打退化成链表

### 正文

**① 先分清「是不是哈希」**

| 容器                      | 底层结构             | 查找              | 遍历顺序     |
| ----------------------- | ---------------- | --------------- | -------- |
| `unordered_map`         | 桶数组 + 链表节点（链地址法） | 平均 O(1)，最坏 O(n) | 无序       |
| `map`                   | 红黑树              | O(log n)        | 按 key 升序 |
| `unordered_set` / `set` | 同上，只是没有 value    | 同上              | 同上       |

`std::unordered_map` 之所以必须是节点式（链式），是标准的一条硬约束：**rehash 只让迭代器失效，元素的引用和指针仍然有效**。开放寻址在扩容时要把元素搬进新数组，做不到这一点。

**② 六种写法**

1）最常用：直接声明

```cpp
unordered_map<int, int> mp;
mp[1] = 10;              // key 不存在 → 先插入 {1, 0}，再赋 10
mp.insert({2, 20});      // 已存在则不动，返回 {it, false}
mp.emplace(3, 30);       // 原地构造，省一次 pair 拷贝
```

2）初始化列表

```cpp
unordered_map<string, int> cnt = {{"a", 1}, {"b", 2}};
```

3）自定义 key：需要同时提供相等比较和哈希函数

```cpp
struct Point { int x, y; bool operator==(const Point& o) const { return x == o.x && y == o.y; } };
struct PointHash {
    size_t operator()(const Point& p) const {
        return hash<int>{}(p.x) * 1000003ULL ^ hash<int>{}(p.y);
    }
};
unordered_map<Point, int, PointHash> mp;
```

4）PBDS（GCC 专属，竞赛卡常用）

```cpp
#include <ext/pb_ds/assoc_container.hpp>
using namespace __gnu_pbds;
gp_hash_table<int, int> mp;   // 开放寻址；cc_hash_table 是拉链版
```

比 `unordered_map` 快，但内存占用高，且 MSVC 上没有。

5）手写数组拉链（最贴近原理的写法）

```cpp
const int BUK = 100003;             // 桶数取质数
int head[BUK], nxt[N], key[N], val[N], tot;
void init() { memset(head, -1, sizeof head); tot = 0; }

int& at(int k) {                    // 返回引用，语义等价于 mp[k]
    int h = (k % BUK + BUK) % BUK;
    for (int i = head[h]; i != -1; i = nxt[i])
        if (key[i] == k) return val[i];
    key[tot] = k; val[tot] = 0; nxt[tot] = head[h]; head[h] = tot;
    return val[tot++];
}
```

`head[h]` 是桶 `h` 的链表头，`nxt[i]` 是节点 `i` 的后继，`tot` 是已用节点数。注意 `at()` 返回 `int&`——这正是 `mp[k]` 返回的东西（见下一条）。

6）离散化（不是哈希，但解决同一个「大值域 → 小下标」问题）

```cpp
sort(a.begin(), a.end());
a.erase(unique(a.begin(), a.end()), a.end());
int id = lower_bound(a.begin(), a.end(), x) - a.begin();
```

**③ 怎么选**

- 一般业务：`unordered_map`；规模已知就先 `mp.reserve(n)`
- 只读 / 性能敏感：`absl::flat_hash_map`、`boost::unordered_flat_map`
- 要按 key 有序或做范围查询：`map`
- 竞赛且卡常：`gp_hash_table` 或手写拉链
- 值域大但只当编号用：离散化

**④ 最坏情况：hash flooding**

平均 O(1) 的前提是哈希均匀。若大量 key 落进同一个桶，就退化成链表 O(n)。在服务端这是**可被构造**的攻击面：攻击者挑一批哈希值相同的字符串，把表打成链表。对策是**带随机种子的哈希**——`absl` / `folly` 默认这么做，标准库不保证。标准库的 `std::hash<std::string>` 在 libstdc++ 与 MSVC 上都是确定性的，不随机化。

### 自测

- [ ] 为什么 `std::unordered_map` 不能用开放寻址实现？
- [ ] 手写拉链里 `head[]`、`nxt[]`、`tot` 各自的作用是什么？插入为什么用头插？
- [ ] 已知要存 10^6 个数，插入前该调用哪个函数、为什么？

## `mp->second` 为什么编译不过？`unordered_map` 的访问方式 #cpp #stl

> **速查**
> - `mp` 是对象、不是指针；`unordered_map` 本身也没有 `second` 成员 → `mp->second` 必错
> - `mp[k]` 返回 `V&`（这里是 `int&`），**它本身就是值**，再取 `.second` 是错上加错
> - `second` 只属于 `pair`；迭代器解引用才是 pair → `mp.find(k)->second`
> - `mp[k]` 查不到会**插入** `{k, V()}`；只查不改用 `find` / `count` / C++20 `contains`
> - `->` 左边必须是迭代器或指针；map 指针取值得写 `(*pm)[k]`

### 正文

**① 对照表**

| 写法 | 结果类型 | 说明 |
|---|---|---|
| `mp[k]` | `V&` | 不存在则插入 `V()` |
| `mp.at(k)` | `V&` | 不存在抛 `std::out_of_range` |
| `mp.find(k)` | `iterator` | 找不到返回 `mp.end()` |
| `mp.find(k)->second` | `V&` | 迭代器指向 `pair<const K, V>` |
| `mp[k].second` | 编译错误 | `V = int` 没有 `second` |
| `mp->second` | 编译错误 | `unordered_map` 没有 `second` |
| `(*pm)[k]` | `V&` | `pm` 的类型是 `unordered_map*` 时 |

**② 错在哪**

`->` 的含义是「取左边所指对象的成员」。`mp` 是栈上对象，类型是 `unordered_map<int,int>`，成员只有 `begin` / `end` / `find` / `size` / `operator[]` 这些，没有 `second`。报错形如：

```
error: no member named 'second' in 'std::unordered_map<int, int>'
```

`second` 唯一来自 `std::pair`，而 map 的 `value_type` 恰好是 `pair<const K, V>`。所以必须先拿到 pair：要么拿到迭代器再 `->`，要么在 range-for 里拿元素。

```cpp
for (auto& p : mp)      use(p.second);   // p: pair<const int,int>&
for (auto& [k, v] : mp) use(v);          // C++17 结构化绑定
auto it = mp.find(1);
if (it != mp.end())     use(it->second);
```

**③ 什么时候 `.second` 是对的**

取决于 `V` 是什么。`mp[k]` 的类型是 `V&`：

```cpp
unordered_map<int, int>           a;  // V = int   → a[1] 就是值
unordered_map<int, pair<int,int>> b;  // V = pair  → b[1].second 合法
b[1] = {2, 3};
cout << b[1].first << b[1].second;    // 输出 23
```

一句话：**`[]` 剥掉的是 map 这一层，不是 pair 这一层。**

**④ 五个必踩的坑**

1）`mp[k]` 是「查 + 插」，不是纯查

```cpp
unordered_map<int,int> mp;
if (mp[1] == 0) {}        // 哪怕只是判断，也插入了 {1, 0}
cout << mp.size();        // 1
```

纯查询：

```cpp
auto it = mp.find(1);
if (it != mp.end() && it->second == 0) { /* ... */ }
```

2）边遍历边删

```cpp
for (auto it = mp.begin(); it != mp.end(); )
    if (bad(it->second)) it = mp.erase(it);   // erase 返回后继
    else                 ++it;
```

`it = mp.erase(it)` 是唯一正确的写法；先缓存 `end()` 再 `erase` 属于未定义行为。

3）rehash 让迭代器失效，但**引用 / 指针不失效**

```cpp
auto& v = mp[1];
mp.reserve(100000);   // 触发 rehash
v = 42;               // ✓ 仍然有效，指向原节点
```

这条是 `unordered_map` 与 `vector` 最大的差别：vector 扩容要搬元素，引用全废；`unordered_map` 只重挂桶链，节点本身不动。

4）已知规模就 `reserve`

```cpp
mp.reserve(n);                // 保证装下 n 个元素不 rehash
mp.max_load_factor(0.7f);     // 可选：桶更稀疏、冲突更少、内存更多
```

默认 `max_load_factor` 是 1.0，`reserve(n)` 保证桶数 ≥ n。

5）`size()` 是 `size_t`

```cpp
for (int i = 0; i < mp.size(); ++i)   // 有符号 / 无符号比较警告
if (mp.size() - 1 >= 0)               // 空表时下溢成巨大值，恒为真
```

### 自测

- [ ] `mp[1]` 和 `mp.at(1)` 在 key 不存在时分别发生什么？
- [ ] `unordered_map` rehash 后，为什么 `auto& v = mp[1]` 仍然可用，而 `vector` 的引用不行？
- [ ] 遍历删除时为什么必须写 `it = mp.erase(it)`？

## `unordered_map` 怎么用？从声明到遍历的完整流程 #cpp #stl

> **速查**
> - 增：`mp[k] = v` / `insert` / `emplace`；查：`find` / `count` / `contains`；删：`erase`
> - 计数写 `++mp[w]`：key 不存在时**先插入 0 再自增**——便利，也是坑的来源
> - 遍历 `for (auto& [k, v] : mp)`；边遍历边删只能 `it = mp.erase(it)`
> - 按 value 排序要先灌进 `vector<pair<K,V>>` 再 `sort`，不能直接排 map
> - 规模已知先 `reserve`；只问「在不在」用 `unordered_set`，要顺序用 `map`

### 正文

**① 四个动作，一张表**

| 动作 | 写法 | key 不存在时 |
|---|---|---|
| 赋值 / 插入 | `mp[k] = v` | 插入 `{k, v}` |
| 插入 | `mp.insert({k, v})` / `mp.emplace(k, v)` | 插入；已存在则不动 |
| 查 | `mp.find(k)` / `mp.count(k)` / `mp.contains(k)`（C++20） | 返回 `end()` / 0 / `false`，不插入 |
| 删 | `mp.erase(k)` / `mp.erase(it)` | 返回删除个数 / 后继迭代器 |

**② 一个能跑的最小例子**

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    vector<string> words = {"a", "b", "a", "c", "a"};
    unordered_map<string, int> cnt;
    cnt.reserve(words.size());              // 规模已知，先预留，避免多次 rehash

    for (auto& w : words) ++cnt[w];         // 计数：不存在时插入 0，再自增

    for (auto& [w, c] : cnt)                // 遍历（顺序不定）
        cout << w << " : " << c << '\n';

    auto it = cnt.find("z");                // 纯查询，不插入
    if (it == cnt.end()) cout << "z 不存在\n";

    cnt.erase("b");                         // 按 key 删

    vector<pair<string, int>> v(cnt.begin(), cnt.end());   // 按频次降序
    sort(v.begin(), v.end(),
         [](const auto& x, const auto& y) { return x.second > y.second; });
}
```

**③ 三个高频场景**

1）频率统计

```cpp
unordered_map<char, int> freq;
for (char c : s) ++freq[c];
```

2）邻接表 / 按名字分组

```cpp
unordered_map<string, vector<int>> g;
g[name].push_back(id);      // 等价于先建 vector 再 push
```

3）记忆化

```cpp
unordered_map<State, int> memo;
int solve(State s) {
    auto it = memo.find(s);
    if (it != memo.end()) return it->second;
    int ans = /* 递归计算 */ 0;
    memo[s] = ans;          // 此时 s 一定不存在，用 [] 安全
    return ans;
}
```

**④ 选容器的判据**

- 只问「在不在」→ `unordered_set`
- 要 key 有序，或要做 `lower_bound` 这类范围查询 → `map`
- key 是 `0..n` 的连续小整数 → 直接用 `vector`，哈希表在这里只有额外开销
- 需要自定义 key → 见上一条的 `PointHash` 写法

### 自测

- [ ] `++cnt[w]` 在 `w` 第一次出现时为什么也能正确工作？
- [ ] `mp.insert({k, v})` 和 `mp[k] = v` 在 key 已存在时的行为有什么不同？
- [ ] 想按频次从高到低输出，为什么不能直接 `sort(mp.begin(), mp.end())`？

## `unordered_set` 怎么用？ #cpp #stl

> **速查**（只讲用法）
> - 声明 `unordered_set<int> st;`；增 `st.insert(x)`；删 `st.erase(x)`；查 `st.count(x)` 或 `st.contains(x)`
> - **没有 `st[i]`**——set 只有 key，没有 value
> - `insert` 返回 `pair<iterator, bool>`：那个 `bool` 就是「是否真的插进去了」，判重一次搞定
> - 遍历 `for (int x : st)`，**顺序不定**；要升序用 `set`
> - 去重：`unordered_set<int> s(v.begin(), v.end());` 然后 `s.size()`

### 正文

**① 五个动作**

| 目的 | 写法 |
|---|---|
| 声明 | `unordered_set<string> st;` |
| 插入 | `st.insert(x)` / `st.emplace(x)` |
| 删除 | `st.erase(x)` / `st.erase(it)` |
| 查询 | `st.count(x)`（0 或 1）/ `st.contains(x)`（C++20） |
| 大小 / 判空 | `st.size()` / `st.empty()` |

**② 最常用的一招：用返回值同时判重和插入**

```cpp
unordered_set<int> seen;
for (int x : a) {
    auto [it, ok] = seen.insert(x);   // ok 为 true 表示首次出现
    if (!ok) { /* x 重复 */ }
}
```

比「先 `count` 再 `insert`」少一次查找。只想判重不需要插入时，才用 `count` / `contains`。

**③ 三种去重写法**

```cpp
// 1) vector 去重，不保序
unordered_set<int> s(v.begin(), v.end());
v.assign(s.begin(), s.end());

// 2) 只想知道「有多少个不同元素」
int distinct = unordered_set<int>(v.begin(), v.end()).size();

// 3) 保序去重：按首次出现顺序保留
unordered_set<int> seen;
vector<int> out;
for (int x : v)
    if (seen.insert(x).second) out.push_back(x);
```

**④ 遍历与边遍历边删**

```cpp
for (const auto& x : st) use(x);       // 顺序不定

for (auto it = st.begin(); it != st.end(); )
    if (bad(*it)) it = st.erase(it);   // erase(it) 返回后继
    else          ++it;
```

**⑤ 四个坑**

1）没有 `st[i]`、没有 `st.at(i)`：set 只有 key。需要 key 映射到 value 时用 `unordered_map`。

2）`erase(x)` 返回删除个数（0 或 1）；`erase(it)` 返回后继迭代器。两者别混。

3）`st.count(x)` 的返回值在 `unordered_set` 里只有 0 / 1；如果看到有人用 `multiset` 的 `count` 语义来理解它，会误判。

4）自定义类型要做 key，仍然要提供相等比较和哈希函数，写法和 `unordered_map` 的 `PointHash` 完全一样：

```cpp
struct Point { int x, y; bool operator==(const Point& o) const { return x == o.x && y == o.y; } };
struct PointHash {
    size_t operator()(const Point& p) const {
        return hash<int>{}(p.x) * 1000003ULL ^ hash<int>{}(p.y);
    }
};
unordered_set<Point, PointHash> st;
```

**⑥ 和 `vector` 怎么选**

元素个数只有几十个时，`vector` 加 `find` 往往比 `unordered_set` 更快——连续内存、无哈希计算、无节点分配。`unordered_set` 的优势要到元素数量上千才开始明显。

### 自测

- [ ] 怎样用 `insert` 的返回值一次完成「查重 + 插入」？
- [ ] `unordered_set` 为什么没有 `operator[]`？
- [ ] 想按首次出现顺序去重，代码怎么写？
- [ ] 自定义结构体放进 `unordered_set`，需要提供哪两样东西？
