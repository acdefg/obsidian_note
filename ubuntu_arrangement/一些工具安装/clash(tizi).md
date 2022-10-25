### 1. 安装Clash(搭个tizi，你懂的)

第一步：到 [https://github.com/Dreamacro/clash/releases](https://link.zhihu.com/?target=https%3A//github.com/Dreamacro/clash/releases) 下载最新的 Linux 版 Clash，例如：[clash-linux-amd64-v0.19.0.gz](https://link.zhihu.com/?target=https%3A//github.com/Dreamacro/clash/releases/download/v0.19.0/clash-linux-amd64-v0.19.0.gz)。解压后得到一个可执行文件 clash-linux-amd64-v0.19.0：

```bash
tar -zxvf clash-linux-amd64-v0.19.0.gz
```

第二步：使用 mv 命令移动到 /usr/local/bin/clash：

```bash
sudo mv clash-linux-amd64-v0.19.0 /usr/local/bin/clash
```

第三步：终端输入 sudo chmod +x /usr/local/bin/clash 添加执行权限；

```bash
sudo chmod +x /usr/local/bin/clash
```

第四步：终端执行 clash 命令，运行 clash；

```bash
# 运行 clash
clash
```

此时会在 /home/{用户ID}/.config/clash 目录下生成两个文件：config.yaml 和 Country.mmdb；编辑 config.yaml 文件，配置代理服务器信息和规则，部分商家会提供yaml文件，下载后 copy 过来即可；

重启 clash（关闭并重新打开终端，执行 clash 命令）以加载更新后的配置文件；

保持 clash 运行，打开浏览器访问 clash.razord.top 进行策略配置、选择代理线路等等（可能需要根据提示输入IP、端口和口令，具体内容可在 config.yaml 中查看；

继续保持 clash 运行，在系统网络设置中设置手动代理 **Settings>Network>Network Proxy>Manual（设置>网络>代理>手动）**，配置信息参考 config.yaml 或者启动 clash 时终端输出的日志。此时就可以通过 clash 访问网络了。

> 按照前面的方式配置好后，每次系统启动时都需要打开终端，执行 clash 命令，并且终端不可以关闭，否则整个 clash 进程就结束了。如果不想一直保持终端打开，可使用 nohup clash 命令启动后台运行。或者希望开机自启动 clash，可将 `nohup clash` 这段命令加入到前面提到的 start-service.sh 脚本的**最后**。

## 参考
1. https://zhuanlan.zhihu.com/p/139305626?utm_source=pocket_mylist
2. 