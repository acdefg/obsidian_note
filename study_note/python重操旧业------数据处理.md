重命名表头
```python
df = pd.read_csv('xxx.csv', names=new_columns, header=0)
```

无表头
```python
data = pd.read_csv(filepath,encoding='utf-8',header =None)
print(data.to_string())
```
[python读csv文件时指定行为表头或无表头\_痴迷、淡然\~的博客-CSDN博客](https://blog.csdn.net/qq_36512295/article/details/87930922)
