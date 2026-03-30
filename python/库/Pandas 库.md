Pandas是一个基于numpy的库

# 1.一维数据Series

把它想象成单条走廊，走廊上有很多门，门上有标签，门里面有内容
即excle中的一列
```python
import pandas as pd

# 从列表创建Series 
s1 = pd.Series([10, 20, 30, 40], index=['a', 'b', 'c', 'd']) # 列表也可以用变量替代  
# 从字典创建  
s2 = pd.Series({'a': 10, 'b': 20, 'c': 30, 'd': 40})
```

![[pandas库.png]]
s1和s2的结果完全一致

## **标签索引与位置索引**

.loc 和 .iloc
```python
data = [10, 20, 30, 40]
s1 = pd.Series(data, index=['a', 'b', 'c', 'd'])
# .loc方法可以返回标签对应的数据，不存在的标签即报错，也可以用来更改数据
temp = s1.loc['a'] 
s1.loc['a'] = 100
# 相对的，.iloc是一个通过列表的index来获取数据的方法,也可以来更改数据
print(s1.iloc[0])
```

## pass(想不到名字)

```python
s1 = pd.Series([10, 20, 30, 40, 50], index=['a', 'b', 'c', 'd', "e"])
print(s1[s1 >= 40]) #  这会返回所有的值大于40的Series
```

# 2. DataFrame
DataFrame是由行和列组成的数据，和excle中的多行多列的表格类似
==**index永远是行**==
构造DataFrame
```python
import pandas as pd  
  
data = {  
    "Name": ["Spongebob", "Patrick", "Squidward"],  
    "Age": [30, 35, 50]  
}  
  
df = pd.DataFrame(data, index=['a', 'b', 'c'])  
  
print(df)
```

## DataFrame .loc 和 .iloc的使用

```python
import pandas as pd  
  
data = {  
    "Name": ["Spongebob", "Patrick", "Squidward"],  
    "Age": [30, 35, 50]  
}  
  
df = pd.DataFrame(data, index=['a', 'b', 'c'])  
# .loc可以按行（标签）读取数据，
print(df.loc['a'])
# 同理.iloc可以进行位置索引
print(df.iloc[0])
```

## 增减一个新列

```python
import pandas as pd  
  
data = {  
    "Name": ["Spongebob", "Patrick", "Squidward"],  
    "Age": [30, 35, 50]  
}  
  
df = pd.DataFrame(data, index=['a', 'b', 'c'])
#增加一个新列  
df["Job"] = ["Cook", "N/A", "Cashier"]
#添加新行
new_rows = pd.DataFrame([{"Name": "Sandy", "Age": 28, "Job": "Engineer"},
    {"Name": "Eugene", "Age": 60, "Job": "Manager"}],
    index=["Employee 4", "Employee 5"]) #注意有[]
df = pd.concat([df, new_row])

print(df)
```

# 3. 切片和索引

## 索引

基本语法：`df.iloc[行索引, 列索引]`
- **行索引**：选择哪些行
- **列索引**：选择哪些列
- 可以省略第二个参数，表示选择所有列
如果想省略行，则行的位置用 `:` 来代替

## 切片（索引的特殊形式）

.iloc这个切片和python的原生切片都是左闭右开
行索引 -> 行起始:行结束:行步长
列索引 -> 列起始:列结束:列步长
基本语法：`df.iloc[行起始:行结束:行步长, 列起始:列结束:列步长]`

**规则总结**：
- **左区间不写**：默认从 `0` 开始
- **右区间不写**：默认到末尾（包含最后一行/列）
- **步长不写**：默认为 `1`
- **步长为 `-1`**：反向切片（注意起始和结束的顺序）

文字版：iloc\[左区间:右区间:step(行切片),左区间:右区间:step(列切片)]（:step可以不写，step为-1时为反向切片，左区间不写默认为0，右区间不写默认结束）

==因为:step可以不写，因此切片只有一个:默认为不是步长的那个冒号==

# 4.CSV的读取
