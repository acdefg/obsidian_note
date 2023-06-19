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


```python
# 设置某一行为列索引【表头】
c_list = df.values.tolist()[1]  # 得到想要设置为列索引【表头】的某一行提取出来
df.columns = c_list  		    # 设置列索引【表头】
df.drop([1], inplace=True)  	# 将原来的那一行删掉。
								# 这里的inplace=True，表示就在df这个数据表中进行修改，默认是False
print(df)
    2  x  h wd
0  1  z  x  e
2  3  f  j  x
3  4  k  s  j

# df_new = df.drop([1])	# 如果是默认的话，做出的修改就要将修改后的内容赋予新的变量才能呈现出和上面一样的效果
# print(df_new)

```
[pandas.DataFrame设置某一行为表头（列索引），设置某一列为行索引，按索引取多行多列\_pandas第一行设置为表头\_Admiral\~的博客-CSDN博客](https://blog.csdn.net/Admiral_x/article/details/126415277)

[Python excel 合并居中值相同的单元格\_python合并相同内容单元格\_一根咸鱼\_的博客-CSDN博客](https://blog.csdn.net/weixin_47597129/article/details/124319638)
