#### modelsim直接仿真testbrench文件步骤
1. 打开modelsim，新建一个工程
![300](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20220921183145.png)
2. 填写名称和路径
![300](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20220921183453.png)
3. 没有现成文件选creat，有的话选add
![300](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20220921183536.png)
4. 创建完成后，编写程序
![500](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20220921183749.png)
5. 右键左边栏的文件，选择complie编译
![300](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20220921183920.png)
6. 切换到libraries，打开work，可以看到文件（这个是我之前的，如果没有的话，去project那里选中添加到当前项目，或者关掉modelsim重启一下）
![300](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20220921184028.png)
7. 右键编译，仿真，按步骤，1没有问题后点2
![300](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20220921184303.png)
8. 在新出现的object选项卡里面右键
![500](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20220921184425.png)
可以在wave界面中得到
![300](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20220921184506.png)
9. 在最下方的界面输入run 50ns（根据作业程序可输入50-100，不能输100）
![](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20220921184645.png)
就会有波形，调整到合适大小即可
10.修改界面颜色
参考链接：[Modelsim修改波形显示颜色_hlc0015的博客-CSDN博客_modelsim波形背景颜色](https://blog.csdn.net/qq_15062763/article/details/104422961)
![400](https://raw.githubusercontent.com/acdefg/cdn/main/obsidian/20220921184917.png)
