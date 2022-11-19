[gcc编译优化-O0 -O1 -O2 -O3 -OS解析_奔跑的码农的博客-CSDN博客_gcc o2](https://blog.csdn.net/wuxing26jiayou/article/details/96132721) 选项内容具体描述
[汇编视角:不同优化级别下的GCC行为分析 | 海森的博客](https://hisenz.com/post/%E6%B1%87%E7%BC%96%E8%A7%86%E8%A7%92-%E4%B8%8D%E5%90%8C%E4%BC%98%E5%8C%96%E7%BA%A7%E5%88%AB%E4%B8%8B%E7%9A%84GCC%E8%A1%8C%E4%B8%BA%E5%88%86%E6%9E%90/) 👍

想使用反汇编的方式查看不同选项对于程序的优化，于是想到了另一门课学到的交叉编译，可以生成 arm64 位环境下的.o 文件，再用.o 反汇编生成汇编码，对比查看不同选项在汇编角度的优化。
选择 memcpy 作为例子是因为它的实现代码简单, 但是涉及了传参, 条件判断和循环, 是逻辑密集型的代码, 能很好的体现 gcc 在逻辑上的优化。
![500](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202211171818644.png)

```C
// This is the implementation in coreutils  
  
// Since no stdlib included, define size_t  
// On x64 size_t is defined as unsigned long  
  
#define size_t unsigned long	  
void * memcpy (void *destaddr, void const *srcaddr, size_t len)  
{  
    char *dest = destaddr;  
    char const *src = srcaddr;  
  
    while (len-- > 0)  
        *dest++ = *src++;  
    return destaddr;  
}
```

使用交叉编译，编译为 arm64 位环境的程序进行测试, 因为程序中没有 main 函数所以在交叉编译时加上 -c 选项：
```shell
aarch64-linux-gnu-gcc -c -o memcy0.o memcy.c -O0
```
使用 file 指令打印文件类型，可以看到该程序属于在 arm64 环境：
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202211171841155.png)
使用 `objdump ` 对程序进行反汇编：
`aarch64-linux-gnu-objdump -d memcy0.o `
阅读代码，添加了一下注释
```asm
memcy0.o:     file format elf64-littleaarch64

Disassembly of section .text:

0000000000000000 <memcpy>:
   0:	d100c3ff 	sub	sp, sp, #0x30   ;压栈
   4:	f9000fe0 	str	x0, [sp, #24]   ;传递第一个参数(指针地址)
   8:	f9000be1 	str	x1, [sp, #16]   ;传递第二个参数(指针地址)
   c:	f90007e2 	str	x2, [sp, #8]    ;传递第三个参数
  10:	f9400fe0 	ldr	x0, [sp, #24]   ;x0  --> *destaddr 指针地址复制 
  14:	f90013e0 	str	x0, [sp, #32]   ;store *dest
  18:	f9400be0 	ldr	x0, [sp, #16]   ;x0  --> *src 指针地址复制
  1c:	f90017e0 	str	x0, [sp, #40]   ;store *src
  20:	14000009 	b	44 <memcpy+0x44> ;跳转到44，执行len的递减
  24:	f94017e1 	ldr	x1, [sp, #40]    ;loop start  x1 --> src (address)
  28:	91000420 	add	x0, x1, #0x1     ;x0--->src + 1
  2c:	f90017e0 	str	x0, [sp, #40]    ;src = src + 1
  30:	f94013e0 	ldr	x0, [sp, #32]    ;x0 --> *dest (address)
  34:	91000402 	add	x2, x0, #0x1     ;x2--->dest + 1
  38:	f90013e2 	str	x2, [sp, #32]    ;dest = dest + 1
  3c:	39400021 	ldrb	w1, [x1]     ;w1 --> *src (data)
  40:	39000001 	strb	w1, [x0]     ;*dest --> w1 (data)
  44:	f94007e0 	ldr	x0, [sp, #8]     ;读入len
  48:	d1000401 	sub	x1, x0, #0x1     ;len-1
  4c:	f90007e1 	str	x1, [sp, #8]     ;len = len -1
  50:	f100001f 	cmp	x0, #0x0         ;len >0 ?
  54:	54fffe81 	b.ne	24 <memcpy+0x24>  // b.any  
  58:	f9400fe0 	ldr	x0, [sp, #24]
  5c:	9100c3ff 	add	sp, sp, #0x30
  60:	d65f03c0 	ret                  ;return
```

```asm
memcy1.o:     file format elf64-littleaarch64

Disassembly of section .text:

0000000000000000 <memcpy>:
   0:	b40000e2 	cbz	x2, 1c <memcpy+0x1c>     ;x2 = 0 --> ret; x2 != 0 --> continue 
   4:	d2800003 	mov	x3, #0x0                   	// #0  ; x3 = 0  i
   8:	38636824 	ldrb	w4, [x1, x3]         ;w4 --> *(src+i) 
   c:	38236804 	strb	w4, [x0, x3]         ;*(dest+i)  --> w4
  10:	91000463 	add	x3, x3, #0x1             ;i = i + 1
  14:	eb02007f 	cmp	x3, x2                   ;if i < x2, loop
  18:	54ffff81 	b.ne	8 <memcpy+0x8>  // b.any
  1c:	d65f03c0 	ret
```

```shell
memcy2.o:     file format elf64-littleaarch64

Disassembly of section .text:

0000000000000000 <memcpy>:
   0:	b40000e2 	cbz	x2, 1c <memcpy+0x1c>
   4:	d2800003 	mov	x3, #0x0                   	// #0
   8:	38636824 	ldrb	w4, [x1, x3]
   c:	38236804 	strb	w4, [x0, x3]
  10:	91000463 	add	x3, x3, #0x1
  14:	eb02007f 	cmp	x3, x2
  18:	54ffff81 	b.ne	8 <memcpy+0x8>  // b.any
  1c:	d65f03c0 	ret

```

```shell
memcy3.o:     file format elf64-littleaarch64

Disassembly of section .text:

0000000000000000 <memcpy>:
   0:	d1000444 	sub	x4, x2, #0x1
   4:	b4000762 	cbz	x2, f0 <memcpy+0xf0>
   8:	91000423 	add	x3, x1, #0x1
   c:	cb030003 	sub	x3, x0, x3
  10:	f100387f 	cmp	x3, #0xe
  14:	fa468880 	ccmp	x4, #0x6, #0x0, hi  // hi = pmore
  18:	540006e9 	b.ls	f4 <memcpy+0xf4>  // b.plast
  1c:	f100389f 	cmp	x4, #0xe
  20:	54000789 	b.ls	110 <memcpy+0x110>  // b.plast
  24:	927cec45 	and	x5, x2, #0xfffffffffffffff0
  28:	d2800003 	mov	x3, #0x0                   	// #0
  2c:	d503201f 	nop
  30:	3ce36820 	ldr	q0, [x1, x3]
  34:	3ca36800 	str	q0, [x0, x3]
  38:	91004063 	add	x3, x3, #0x10
  3c:	eb05007f 	cmp	x3, x5
  40:	54ffff81 	b.ne	30 <memcpy+0x30>  // b.any
  44:	927cec43 	and	x3, x2, #0xfffffffffffffff0
  48:	cb030084 	sub	x4, x4, x3
  4c:	8b030005 	add	x5, x0, x3
  50:	8b030026 	add	x6, x1, x3
  54:	eb03005f 	cmp	x2, x3
  58:	540004c0 	b.eq	f0 <memcpy+0xf0>  // b.none
  5c:	cb030042 	sub	x2, x2, x3
  60:	d1000447 	sub	x7, x2, #0x1
  64:	f10018ff 	cmp	x7, #0x6
  68:	54000129 	b.ls	8c <memcpy+0x8c>  // b.plast
  6c:	fc636820 	ldr	d0, [x1, x3]
  70:	927df041 	and	x1, x2, #0xfffffffffffffff8
  74:	8b0100a5 	add	x5, x5, x1
  78:	8b0100c6 	add	x6, x6, x1
  7c:	cb010084 	sub	x4, x4, x1
  80:	fc236800 	str	d0, [x0, x3]
  84:	eb01005f 	cmp	x2, x1
  88:	54000340 	b.eq	f0 <memcpy+0xf0>  // b.none
  8c:	394000c1 	ldrb	w1, [x6]
  90:	390000a1 	strb	w1, [x5]
  94:	b40002e4 	cbz	x4, f0 <memcpy+0xf0>
  98:	394004c1 	ldrb	w1, [x6, #1]
  9c:	390004a1 	strb	w1, [x5, #1]
  a0:	f100049f 	cmp	x4, #0x1
  a4:	54000260 	b.eq	f0 <memcpy+0xf0>  // b.none
  a8:	394008c1 	ldrb	w1, [x6, #2]
  ac:	390008a1 	strb	w1, [x5, #2]
  b0:	f100089f 	cmp	x4, #0x2
  b4:	540001e0 	b.eq	f0 <memcpy+0xf0>  // b.none
  b8:	39400cc1 	ldrb	w1, [x6, #3]
  bc:	39000ca1 	strb	w1, [x5, #3]
  c0:	f1000c9f 	cmp	x4, #0x3
  c4:	54000160 	b.eq	f0 <memcpy+0xf0>  // b.none
  c8:	394010c1 	ldrb	w1, [x6, #4]
  cc:	390010a1 	strb	w1, [x5, #4]
  d0:	f100109f 	cmp	x4, #0x4
  d4:	540000e0 	b.eq	f0 <memcpy+0xf0>  // b.none
  d8:	394014c1 	ldrb	w1, [x6, #5]
  dc:	390014a1 	strb	w1, [x5, #5]
  e0:	f100149f 	cmp	x4, #0x5
  e4:	54000060 	b.eq	f0 <memcpy+0xf0>  // b.none
  e8:	394018c1 	ldrb	w1, [x6, #6]
  ec:	390018a1 	strb	w1, [x5, #6]
  f0:	d65f03c0 	ret
  f4:	d2800003 	mov	x3, #0x0           ;loop part       	// #0
  f8:	38636824 	ldrb	w4, [x1, x3]
  fc:	38236804 	strb	w4, [x0, x3]
 100:	91000463 	add	x3, x3, #0x1
 104:	eb03005f 	cmp	x2, x3
 108:	54ffff81 	b.ne	f8 <memcpy+0xf8>  // b.any
 10c:	d65f03c0 	ret
 110:	aa0103e6 	mov	x6, x1
 114:	aa0003e5 	mov	x5, x0
 118:	d2800003 	mov	x3, #0x0                   	// #0
 11c:	17ffffd4 	b	6c <memcpy+0x6c>
```

![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202211172105322.png)

```C
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

#define size_t unsigned long 
	
int main(){
    for (size_t i = 0; i<1000000; i++)
    {
        char* str = (char*) malloc(sizeof(char)*1024);
        strcpy(str,"abcdefg");

        char* new_str = (char*) malloc(sizeof(char)*1024);
        memcpy(new_str,str,1024);	// customized memcpy() here
    }

    return 0;
}
	
```


```shell
aarch64-linux-gnu-gcc -o memcpy0.o memcpy_m.c -O0 -static
aarch64-linux-gnu-gcc -o memcpy1.o memcpy_m.c -O1 -static
aarch64-linux-gnu-gcc -o memcpy2.o memcpy_m.c -O2 -static
aarch64-linux-gnu-gcc -o memcpy3.o memcpy_m.c -O3 -static
```

使用 `time` 测试运行时间：
| Options | user/s | system|
| ------- | ---- | ------ |
| O0      |  0.37    |  0.5      |
| O1      |   0.35   |  0.5      |
| O2      |    0.33  |   0.54     |
| O3        |     0.24    |  0.5     |

real：实际时间，从 command 命令行开始执行到运行终止的消逝时间；

user：用户CPU时间，命令执行完成花费的用户CPU时间，即命令在用户态中执行时间总和；

system：系统CPU时间，命令执行完成花费的系统CPU时间，即命令在核心态中执行时间总和；

![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/202211172132126.png)


