---
title: zuoye3
tags: ["note"]
created: 星期日, 六月 28日 2026, 2:58:15 下午
modified: 星期日, 六月 28日 2026, 5:17:00 下午
---

# 踩坑记录

# 使用 DCD 等分配代码得到的地址不对
用了这段代码
```armasm
		AREA Buf, DATA, READWRITE ;
Array 	DCD 0x11, 0x22, 0x33, 0x44 ;
		DCD 0x55, 0x66, 0x77, 0x88
		DCD 0x00, 0x00, 0x00, 0x00
		AREA Example, CODE, READONLY
		ENTRY
		CODE32	
			
			LDR R0, = Array ;
			;MOV R0, #36 ; can't understand why i need give the address individually
			LDR R2, [R0] ;
			MOV R1, #4
			LDR R3, [R0, R1, LSL #2] ;
			ADD R2, R2, R3 ;R2 + R3 ? R2
			MOV R1, #8 ;R1 = 8
			STR R2, [R0, R1, LSL #2] ;
	END
```
读入 Array 的时候一直 R0 的地址都是 0x40000000，始终读不到 Array 真正存储的值，试了一下午吧
最后把这里原来写的 0x40000000 删掉了就可以了

工程初始化设置参考：[Keil下ARM汇编程序建立与调试简介_朝辞暮见的博客-CSDN博客](https://blog.csdn.net/weixin_42048417/article/details/80585993)

> 不同芯片的设置都不一样，以上适用于 s3c2410

不要加初始文件 s3c2410A.s

   另外：ARM 的 M 系列主要用 Thumb 指令，ARM9 和 A 系列主要用 ARM 指令

     S3C2440.S 启动代码中根本就没用 Thumb 指令。

# 打印输出的方法
     
`int 21h` 输出打印，范例如下：
```Assembly
data segment

data ends

code segment
	assume cs:code,ds:data
start:
mov ax,data
mov ds,ax

mov dl,48h;把要输出的内容送入到dl中
mov ah,02 ;表示打印dl中的内容
int 21h

mov dl,10 ;表示打印一个换行符
int 21h


mov dl,03
int 21h

mov ah,4ch ; 使得程序结束
int 21h


code ends
end start

```
参考：[汇编之简单地打印内容_久许的博客-CSDN博客](https://blog.csdn.net/jiuweideqixu/article/details/100562637)

# arm 模式和 thumb 模式的转换
从 arm 转 thumb：
```arm
	AREA Arm_to_Thumb,CODE, READONLY 
	ENTRY 
	CODE32 
start ldr r0,=aaa+1 
	   mov r3,#18 
	   bx r0 CODE16 
aaa  mov r1,#12 
	 mov r2,#10 
	 END
```
关键语句：
```txt
ldr r6,=label
bx r6
```
aaa = label 替换成 thumb 段 label, 前面的寄存器可以是任意寄存器
+1 是为了告诉编译器跳转的是 thumb 代码，这由 bx 的语法决定

从 thumb 转 arm：
```txt
	AREA Arm_to_Thumb,CODE, READONLY 
	ENTRY 
	CODE16 
start 
	ldr r0,=zhangsan 
	mov r3,#18 
	bx r0 
	CODE32 
zhangsan 
	mov r1,#12 
	mov r2,#10 
	END
```
这里不用 +1 是因为末位为偶数位就可以实现跳转 arm
