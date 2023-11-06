在 Ubuntu 22.04 上安装 Clang 编译器非常简单。您可以根据以下步骤进行操作：

1. **使用 APT 软件包管理器安装 Clang**：
    - 打开终端，可以使用快捷键 `Ctrl+Alt+T`。
    - 执行以下命令来更新系统软件包并刷新 APT 软件包列表：
        ```bash
        sudo apt update && sudo apt upgrade
        ```
    - 然后，使用以下命令安装 Clang：
        ```bash
        sudo apt install clang
        ```
2. **检查 Clang 版本**：
    - 您可以运行以下命令来检查已安装的 Clang 版本：
        ```bash
        clang --version
        ```
3. **设置 Clang 版本为默认编译器**（可选）：
    - 默认情况下，系统不会将最新版本的 Clang 设置为全局编译器。
    - 如果您想要将 Clang 设置为默认版本，可以使用以下命令（将 `16` 替换为您系统上已安装的版本）：
        ```bash
        sudo update-alternatives --install /usr/bin/clang++ clang++ /usr/bin/clang++-16 100
        sudo update-alternatives --install /usr/bin/clang clang /usr/bin/clang-16 100
        ```
    - 如果您想要配置使用 Clang 而不是 GCC 的 `make` 实用程序，可以运行以下命令：
        ```bash
        sudo update-alternatives --config cc
        ```
4. **验证安装**：
    - 现在，您可以再次运行 `clang --version` 来确认您想要的版本已经安装在系统上。
示例：创建一个简单的 C 程序并使用 Clang 编译它：
1. 创建一个名为 `hello.c` 的新文件，内容如下：
    ```c
    #include <stdio.h>
    
    int main() {
        printf("Hello, World!\n");
        return 0;
    }
    ```
2. 使用以下命令编译 C 程序：
    ```bash
    clang hello.c -o hello
    ```