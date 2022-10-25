ubuntu版参考：https://developer.aliyun.com/article/760687
以下为上面链接提取出来的部分
- 自动安装
```shell
sudo apt update
sudo apt install nodejs npm
```
这个自动安装nodejs为10.X，npm为6.14.4
查询版本代码为：
```
nodejs --version
npm --version
```
- 指定版本
```
curl -sL https://deb.nodesource.com/setup_14.x | sudo -E bash -
sudo apt-get install -y nodejs
```
将14.x改成任意版本即可
第一句添加源，第二句安装
如没有安装curl，就像我，会报错，按照提示安装即可
```
sudo apt install curl
```

提示：(没处理完，gcc已安装)
```
## Run `sudo apt-get install -y nodejs` to install Node.js 16.x and npm
## You may also need development tools to build native addons:
     sudo apt-get install gcc g++ make
## To install the Yarn package manager, run:
     curl -sL https://dl.yarnpkg.com/debian/pubkey.gpg | gpg --dearmor | sudo tee /usr/share/keyrings/yarnkey.gpg >/dev/null
     echo "deb [signed-by=/usr/share/keyrings/yarnkey.gpg] https://dl.yarnpkg.com/debian stable main" | sudo tee /etc/apt/sources.list.d/yarn.list
     sudo apt-get update && sudo apt-get install yarn
```
- nvm管理多个安装版本(没弄，暂时没必要)