### 仿真说明
编译仿真环境：SEGGER
使用芯片：STM32F103VE
仿真程序：流水灯

### 程序源码
~~~c
#include "stm32f10x.h"
//---------------------------------------------------------------------------
int main(void)
{  

  //SystemInit();

  /* GPIOD Periph clock enable */
  RCC_APB2PeriphClockCmd(RCC_APB2Periph_GPIOC, ENABLE);

  /* Configure PC6 in output pushpull mode */
  GPIO_InitTypeDef GPIO_InitStructure;
  GPIO_InitStructure.GPIO_Pin =  GPIO_Pin_6;
  GPIO_InitStructure.GPIO_Speed = GPIO_Speed_10MHz;
  GPIO_InitStructure.GPIO_Mode = GPIO_Mode_Out_PP;
  GPIO_Init(GPIOC, &GPIO_InitStructure);

// ESE test!
long aa;
while(1){
          /* Set PC6 */
          GPIOC->BSRR = 0x00000040;
          aa=0X1FFFFF;
          while(aa--);

          /* Reset PC6 */
          GPIOC->BRR  = 0x00000040; 
          aa=0X1FFFFF;
          while(aa--);
        }//while

        return 0;
}//main
~~~

### 遇到的问题
#### 1、下载在线包时，加载不出来
一开始使用segger的package manager的时候，只能加载出来已经安装的包，无法下载在线包，查看segger的使用手册，发现这一段
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/Pasted%20image%2020220913112616.png)
发现应该是网络问题
>解决：重复刷新，等待一段时间后官方提供的包就能都显示出来了
#### 2、不能使用外设库
要进行32的仿真，还需要手动添加外设库，通过搜索发现有两种途径可以下载：
1、官方网站：[STM32标准外设软件库 - STMicroelectronics](https://www.stmicroelectronics.com.cn/zh/embedded-software/stm32-standard-peripheral-libraries.html?querycriteria=productId=LN1939)
需要填写邮箱，从邮箱打开下载
2、 STM社区：[意法半导体STM32/STM8技术社区 - 提供最新的ST资讯和技术交流](https://www.stmcu.org.cn/)
需要账户
下载完成后，需要将外设文件中的library复制到studio中，地点可以自由选择(最好保证英文)，之后在project的options中添加该路径，就可以编译成功了。
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/Pasted%20image%2020220913212233.png)
>参考博客：[STM32官方固件库（标准固件库）下载及介绍_csdndgq的博客-CSDN博客_stm32官方库](https://blog.csdn.net/cbkdgq/article/details/88076843)
### 3、printf的输出找不到
printf()的输出会显示到左下角（默认情况）debug_terminal窗口，但是如果使用J_LINK仿真就不会自动显示到那里，上网搜索可以用以下办法
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/Pasted%20image%2020220913103534.png)
如果创立工程的时候没有选，则可以在侧边栏选择当前project，右键options->DEBUG->debugger->Target Connection改成Simulator
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/Pasted%20image%2020220913210941.png)
library->library I/O选择RTT
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/Pasted%20image%2020220913211129.png)
官网还给出了一种解决办法：[RTT in Embedded Studio - SEGGER Wiki](https://wiki.segger.com/RTT_in_Embedded_Studio)

### summary
之前编写32程序都是在keil上面，这次想学一下新软件SEGGER，segger总的来说界面和操作更加简洁美观一些，但是在网上现存的教程比较少，很多问题在外文社区的讨论比较多，而keil在网上现存的资料很多，用keil搭建Samsung s3c2140的环境比较方便。
这次练习找到的直接相关问题的答案比较少，只有一些相关问题的解决方案，比如：是segger软件但不是同样的芯片；是同样的问题，但不是在segger上面，可能需要经过多方比较调试，才能得出解决措施。

### 待完成问题
- 没能在segger上面成功安装samsung芯片的环境