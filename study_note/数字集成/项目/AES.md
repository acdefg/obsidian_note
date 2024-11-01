## 算法
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202409091947012.png)

## 硬件
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202409091954789.png)

1、通过 func 信号可配置其工作模式，每次可以处理 16 Byte 的 state 数据通过 func 信号可配置其工作模式，每次可以处理 16 Byte 的 state 数据
2、每次AES ENGINE可处理16 Byte的data，而AES算法产生或所需的data为64 Byte，因此使用AES_BUF将产生或所需的data进行缓存，之后再与Mem_intf进行传输

TODO：PPA 方案

## quiz
### AES 和 BLAKE2B 的区别
AES（Advanced Encryption Standard，高级加密标准）和 Blake2b 都是常见的加密算法，它们之间存在以下一些区别：

**一、算法类型**
-   **AES**：是一种**对称密钥加密算法**，属于分组密码。它将明文分成固定大小的块进行加密操作。
-   **Blake2b**：是一种哈希函数算法。哈希函数主要用于将任意长度的输入数据转换为固定长度的输出，通常用于数据完整性校验、密码存储等。

**二、加密方式**
-   **AES**：使用相同的密钥进行加密和解密。密钥长度可以是 128 位、192 位或 256 位。在加密过程中，通过多轮的置换、替换和线性混合等操作对数据块进行加密。
-   **Blake2b**：不是用于加密通信内容，而是通过对输入数据进行一系列复杂的运算，生成固定长度的哈希值。这个哈希值具有不可逆性，即很难从哈希值反推出原始输入数据。

**三、应用场景**
-   **AES**：广泛应用于对数据进行加密保护，如文件加密、网络通信加密、数据库加密等。确保数据在存储和传输过程中的保密性。
-   **Blake2b**：常用于数据完整性验证、密码存储的哈希处理、数字签名等场景。例如，在存储用户密码时，通常会使用哈希函数对密码进行处理，而不是直接存储明文密码。

**四、性能特点**
-   **AES**：通常具有高效的加密和解密速度，尤其在硬件实现上性能出色。适合对大量数据进行实时加密和解密操作。
-   **Blake2b**：计算速度也很快，并且能够产生高质量的哈希值。在需要快速计算哈希值的场景下表现良好。

### 对称加密
对称加密算法：AES
加密和解密用到的密钥是相同的，这种加密方式加密速度非常快，适合经常发送数据的场合。缺点是密钥的传输比较麻烦。

非对称加密算法
加密和解密用的密钥是不同的，这种加密方式是用数学上的难解问题构造的，通常加密解密的速度比较慢，适合偶尔发送数据的场合。优点是密钥传输方便。常见的非对称加密算法为 RSA、ECC 和 EIGamal。

