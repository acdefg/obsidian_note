down:: [[visual_studio下载 +opengl配置]]
### eigen 将矩阵输出

```c++
std::ofstream input_file("input.txt", std::ios::trunc);

if (input_file.is_open()) {

input_file << "Input (q):\n" << q.transpose() << "\n";

input_file << "Input (A):\n" << A << "\n";

input_file << "Input (b):\n" << b.transpose() << "\n";

input_file.close();

std::cout << "Input File opened successfully. Absolute path: " << std::filesystem::absolute("./input.txt") << "\n";

} else {

std::cerr << "Error: Unable to open file for writing\n";

}
```
