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
2. [优质SS/SSR/V2Ray/Trojan/Clash机场推荐 | IPLC/IEPL专线加速器梯子推荐 - 选梯子](https://xuantizi.com/best-ss-ssr-v2ray-trojan-xray-proxy.html)

# new
现在ubuntu上有三个clash，
一个在/usr/bin里面，可以直接终端输入clash启动
一个在Template里面，可以进去后输入clash启动
一个在 download 里面，输入./cfw 启动，这个配置了订阅链接（以移动到 Template 年里面了）

分流规则配置（待办）：
https://yattazen.com/tutorial/clash-custom-config.html#:~:text=简易教程 | Clash | 自定义在线分流规则策略组 1 1. 简介,4. 导入配置文件地址到Clash 按照图示导入最终的配置文件到 CFW 或 Clash.Net ，完结撒花！%20
这博客也不错，有空看看

### bug
突然端口号显示为 0，并且上不了网，也改不回来
改了一个更大的 10240, 将就行了
