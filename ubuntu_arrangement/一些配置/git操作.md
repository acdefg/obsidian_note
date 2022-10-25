### 安装和设置git
1. 安装git
```shell
sudo apt-get install git
```
2. 设置账户
```
git config --global user.name "yourname" 
git config --global user.email "youremail"
```
替换的时候不用保留引号
3. 检查
```  
git config user.name 
git config user.email
```
Or

```shell
git config --list
```
会用 vim 打开一个文件，写着上面的信息

4. 生成公匙

```shell
ssh-keygen -t rsa -C "youremail" 
```
提示的地方直接按 Enter

```shell
cat  ~/.ssh/id_rsa.pub
```
或者

```shell
gedit ~/.ssh/id_rsa.pub
```
查看公匙，复制公匙

5. 添加到 github 后台
打开 github 个人主页，点击左上角头像，打开 setting，选择 SSH
![](https://s2.loli.net/2022/05/03/fqpDbIJ81S5ej9W.png)
![](https://s2.loli.net/2022/05/03/McjYFSmEKyhzwg6.png)
添加，确认一下就可以了
