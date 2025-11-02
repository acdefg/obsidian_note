[在ZSH中将新条目添加到PATH变量中](https://qastack.cn/programming/11530090/adding-a-new-entry-to-the-path-variable-in-zsh)
[linux - Adding a new entry to the PATH variable in ZSH - Stack Overflow](https://stackoverflow.com/questions/11530090/adding-a-new-entry-to-the-path-variable-in-zsh/18077919#18077919)

## 方法一
在这里，将此行添加到.zshrc：

export PATH=/home/david/pear/bin:$PATH

`zshconfig` 这个是我改过的打开./zshrc 指令

## 方法二
实际上，使用 ZSH 允许您使用环境变量的特殊映射。因此，您可以简单地执行以下操作：

```
# append
path+=('/home/david/pear/bin')
# or prepend
path=('/home/david/pear/bin' $path)
# export to sub-processes (make it inherited by child processes)
export PATH`
```

对我来说，这是一个非常简洁的功能，可以传播到其他变量。例：
```
typeset -T LD_LIBRARY_PATH ld_library_path :
```


> [!info]
只有方法一成功了，方法二可行性未知

