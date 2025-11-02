[Windows Subsystem for Linux (WSL, Ubuntu) 最新安装教程（2024.11 更新）-CSDN博客](https://blog.csdn.net/wangtcCSDN/article/details/137950545)
# 移动
以前，移动发行版需要手动导出 → 将其作为新发行版导入 WSL → 然后删除原来的发行版，这 3 个步骤。但从 WSL 2.3.11 开始，微软引入了更简单的 `--move` 参数来移动发行版的底层存储。

例如，要将 Ubuntu 22.04 实例移动到 D 盘的一个专用 WSL 文件夹，可以使用以下命令：

列出已安装的发行版：

```
wsl --list
```

将特定发行版移动到指定路径：

```
wsl --manage Ubuntu-22.04 --move <path>
```

移动成功后，会有通知提示

# git clone 问题！