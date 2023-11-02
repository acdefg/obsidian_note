安装指令：
```shell
# 添加 LLVM GPG 密钥
wget -O - https://apt.llvm.org/llvm-snapshot.gpg.key | sudo apt-key add -

# 更新软件包列表
sudo apt-get update

# 添加 LLVM 存储库（根据您的 Ubuntu 版本选择）
# 对于 Ubuntu 18.04：
sudo apt-add-repository "deb http://apt.llvm.org/bionic/ llvm-toolchain-bionic-6.0 main"

# 对于 Ubuntu 20.04：
sudo apt-add-repository "deb http://apt.llvm.org/focal/ llvm-toolchain-focal-6.0 main"

# 对于 Ubuntu 22.04：
sudo apt-add-repository "deb http://apt.llvm.org/jammy/ llvm-toolchain-jammy-6.0 main"

# 安装 Clang 和 lld（链接器）
sudo apt-get install clang-6.0 lld-6.0
```
验证：

```shell
clang++ --version
```
