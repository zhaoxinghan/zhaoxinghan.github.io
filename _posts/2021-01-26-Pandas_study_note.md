---
layout: post
title: Pandas学习笔记 
date: 2021-01-26 10:53
category: Data Mining
author: 
tags: [Data Mining] [pandas]
summary: 
---

# Pandas 学习笔记

* content
{:toc}

Pandas的学习记录



## 数据结构

Pandas有两种数据结构，DataFrame和Series，其中Series是标量数据的集合，DataFrame是一系列Series的集合。

Numpy只有一种数据结构，就是Array。DataFrame可以通过```DataFrame.to_numpy()```转换成Array，转换过程中会损失index属性。

Series具有index属性

````py
In [3]: s = pd.Series([1, 3, 5, np.nan, 6, 8])

In [4]: s
Out[4]: 
0    1.0
1    3.0
2    5.0
3    NaN
4    6.0
5    8.0
dtype: float64
````

DataFrame对象具有index和Columns属性

````py
In [5]: dates = pd.date_range('20130101', periods=6)

In [6]: dates
Out[6]: 
DatetimeIndex(['2013-01-01', '2013-01-02', '2013-01-03', '2013-01-04',
               '2013-01-05', '2013-01-06'],
              dtype='datetime64[ns]', freq='D')

In [7]: df = pd.DataFrame(np.random.randn(6, 4), index=dates, columns=list('ABCD'))

In [8]: df
Out[8]: 
                   A         B         C         D
2013-01-01  0.469112 -0.282863 -1.509059 -1.135632
2013-01-02  1.212112 -0.173215  0.119209 -1.044236
2013-01-03 -0.861849 -2.104569 -0.494929  1.071804
2013-01-04  0.721555 -0.706771 -1.039575  0.271860
2013-01-05 -0.424972  0.567020  0.276232 -1.087401
2013-01-06 -0.673690  0.113648 -1.478427  0.524988
````

## 主要用法

| Index | Function | Description |
| :---: | :--- | --- |
| 1 | .head() | 查看头部数据|
| 2 | .tail(n) | 查看尾部数据 |
|3| .index | 显示索引 |
|4| .columns | 显示列名 |
|5| .describe() | 统计摘要 |
|6| .T | 转置 |
|7| .sort_index(axis=1,ascending=False) | 按轴排序 |
|8| .sort_values(by='B') | 按值排序|
|9| df['A'] | 获取数据 |
|10| df[0:3]或者df['20130102':'20130104'] | 切片行 |
|11| df.loc[dates[0]] | 用标签提取一行数据 |
|12| df.loc[:,['A','B']] | 用标签提取多列数据 |
|13| df.loc['20130102':'20130104',['A','B']] | 用标签切片 |
|14| df.iloc(3) | 用整数位置选择 |
|15| df.iloc[3:5,0:2] or df.iloc[[1,2,4],[0,2]]  | 用整数位置切片 |
|16|  df.iloc[1:3, :] | 整行显示切片 |
|17| df[df.A >0] | 单列值选择 |
|18| df[df > 0] | 选择所有满足条件的值 |
|19| df2[df2['E'].isin(['two', 'four'])] | isin()筛选|

## 赋值

```python

df.at[dates[0], 'A'] = 0

df.iat[0, 1] = 0

df.loc[:, 'D'] = np.array([5] * len(df))

df2[df2 > 0] = -df2

```

## 重建索引

```python

df1= df.reindex(index=dates[0:4],columns=list(df.columns)+['E])
df1.loc[dates[0]:dates[1],'E']=1
```

删除所有含有缺失值的行
```python
df1.dropna(how='any')  # delete all rows contain NaN
df1.fillna(value=5)    # fill Nan with 5
pd.isna(df1)           # get nan mask value
```


## 连接

```py
# 分解为多组
pieces = [df[:3], df[3:7], df[7:]]

pd.concat(pieces)
```

Database style merge

```py
left = pd.DataFrame({'key': ['foo', 'foo'], 'lval': [1, 2]})
right = pd.DataFrame({'key': ['foo', 'foo'], 'rval': [4, 5]})
pd.merge(left, right, on='key')

Out[81]: 
   key  lval  rval
0  foo     1     4
1  foo     1     5
2  foo     2     4
3  foo     2     5
```

## 追加

```python
df = pd.DataFrame(np.random.randn(8, 4), columns=['A', 'B', 'C', 'D'])
s = df.iloc[3]
df.append(s, ignore_index=True)
```

## 分组
```python
 df = pd.DataFrame({'A': ['foo', 'bar', 'foo', 'bar',
  'foo', 'bar', 'foo', 'foo'],
  'B': ['one', 'one', 'two', 'three',
  'two', 'two', 'one', 'three'],
  'C': np.random.randn(8),
  'D': np.random.randn(8)})

df.groupby('A').sum()
Out[93]: 
            C        D
A                     
bar -2.802588  2.42611
foo  3.146492 -0.63958

df.groupby(['A', 'B']).sum()
Out[94]: 
                  C         D
A   B                        
bar one   -1.814470  2.395985
    three -0.595447  0.166599
    two   -0.392670 -0.136473
foo one   -1.195665 -0.616981
    three  1.928123 -1.623033
    two    2.414034  1.600434
```


## 堆叠

stack()与unstack()。压缩后的DataFrame和Series具有多层索引。

## 数据透视表

```python
In [105]: df = pd.DataFrame({'A': ['one', 'one', 'two', 'three'] * 3,
   .....:                    'B': ['A', 'B', 'C'] * 4,
   .....:                    'C': ['foo', 'foo', 'foo', 'bar', 'bar', 'bar'] * 2,
   .....:                    'D': np.random.randn(12),
   .....:                    'E': np.random.randn(12)})
   .....: 

In [106]: df
Out[106]: 
        A  B    C         D         E
0     one  A  foo  1.418757 -0.179666
1     one  B  foo -1.879024  1.291836
2     two  C  foo  0.536826 -0.009614
3   three  A  bar  1.006160  0.392149
4     one  B  bar -0.029716  0.264599
5     one  C  bar -1.146178 -0.057409
6     two  A  foo  0.100900 -1.425638
7   three  B  foo -1.035018  1.024098
8     one  C  foo  0.314665 -0.106062
9     one  A  bar -0.773723  1.824375
10    two  B  bar -1.170653  0.595974
11  three  C  bar  0.648740  1.167115

In [107]: pd.pivot_table(df, values='D', index=['A', 'B'], columns=['C'])
Out[107]: 
C             bar       foo
A     B                    
one   A -0.773723  1.418757
      B -0.029716 -1.879024
      C -1.146178  0.314665
three A  1.006160       NaN
      B       NaN -1.035018
      C  0.648740       NaN
two   A       NaN  0.100900
      B -1.170653       NaN
      C       NaN  0.536826
```

## 时间序列

```py
In [108]: rng = pd.date_range('1/1/2012', periods=100, freq='S')

In [109]: ts = pd.Series(np.random.randint(0, 500, len(rng)), index=rng)

In [110]: ts.resample('5Min').sum()

Out[110]: 
2012-01-01    25083
Freq: 5T, dtype: int64
```

Pandas 函数可以很方便地转换时间段与时间戳。下例把以 11 月为结束年份的季度频率转换为下一季度月末上午 9 点：

```py
In [123]: prng = pd.period_range('1990Q1', '2000Q4', freq='Q-NOV')

In [124]: ts = pd.Series(np.random.randn(len(prng)), prng)

In [125]: ts.index = (prng.asfreq('M', 'e') + 1).asfreq('H', 's') + 9

In [126]: ts.head()
Out[126]: 
1990-03-01 09:00   -0.902937
1990-06-01 09:00    0.068159
1990-09-01 09:00   -0.057873
1990-12-01 09:00   -0.368204
1991-03-01 09:00   -1.144073
Freq: H, dtype: float64
```

## 可视化

DataFrame 的 plot() 方法可以快速绘制所有带标签的列：

```python
In [138]: df = pd.DataFrame(np.random.randn(1000, 4), index=ts.index,
   .....:                   columns=['A', 'B', 'C', 'D'])
   .....: 

In [139]: df = df.cumsum()

In [140]: plt.figure()
Out[140]: <Figure size 640x480 with 0 Axes>

In [141]: df.plot()
Out[141]: <matplotlib.axes._subplots.AxesSubplot at 0x7f2b53a2d7f0>

In [142]: plt.legend(loc='best')
Out[142]: <matplotlib.legend.Legend at 0x7f2b539728d0>
```