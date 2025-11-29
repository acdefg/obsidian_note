查看默认 python 版本

```shell
python  #默认python版本
python3 #python3版本
```

删除原来的 python

```shell
sudo rm -rf /usr/bin/python
```

查看 python3 位置

```shell
whereis python3
```

新建 python3 人软链接

```shell
sudo ln -s /usr/bin/python3.5 /usr/bin/python
```

现在重新查看默认的 python 版本
`python --version` （直接输 python 也可以）

安装 python3

```shell
sudo apt install python3
```
