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
[python 合并内容相同单元格\_梦因you而美的博客-CSDN博客](https://blog.csdn.net/apollo_miracle/article/details/102874716#:~:text=python%20%E5%90%88%E5%B9%B6%E5%86%85%E5%AE%B9%E7%9B%B8%E5%90%8C%E5%8D%95%E5%85%83%E6%A0%BC%201%20from%20openpyxl%20import%20load_workbook%202,type_list%20%3D%20%5B%5D%208%20i%20%3D%202%20%E6%9B%B4%E5%A4%9A%E9%A1%B9%E7%9B%AE)

