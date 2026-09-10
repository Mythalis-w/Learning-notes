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
# 同理.iloc可以进行位置索引，表头不计入位置
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

```python
pd.read_csv("train.csv") #读进来是dataFrame,文件在该目录下

print(train.shape) # (行数, 列数)
print(train.columns) # 列名
print(train.dtypes) # 每列类型
print(train.head()) # 前 5 行

print(train["num_sold"])  #可以读这一列
print(train["num_sold"].head(10)) # 头部10列
print(train["num_sold"].tail(10)) # 尾部10列 

print(train.iloc[0])  # 按位置第 0 行（若索引不是 0 开始，和 loc 可能不同）  
print(train.loc[0]) # 索引为 0 的那一行
```

# 5.时间和日期的提取与规范化

下面出自一个ai模型训练的代码中，我们以此来学习
```python
def add_date_features(df: pd.DataFrame) -> pd.DataFrame:

    out = df.copy()

    dt = pd.to_datetime(out["date"] ''', errors="coerce"''' ) # 将日期转换为datetime类型，这个是pandas用来提取日期的，

    out["year"] = dt.dt.year #提取年份，这里第一个dt是datatime的名字 , .dt是datatime的入口，.year则是提取年份 , `errors="coerce"` 表示异常日期会变成空值（`NaT`）

    out["month"] = dt.dt.month #提取月份，同上

    out["day"] = dt.dt.day #提取日期，同上

    out["dayofweek"] = dt.dt.dayofweek # 提取这一天是星期几，代码自动计算(0-6)

    out["weekofyear"] = dt.dt.isocalendar().week.astype(int) # 提取一年中的第几周（ISO周），并转成整数。  .isocalendar()是转化为ISO日期，年周星期都换  .week是拿周  .astype(int)是转化为int

    out["quarter"] = dt.dt.quarter # 提取季度（1~4）

    out["dayofyear"] = dt.dt.dayofyear # 提取一年中的第几天（1~365/366）

    out["is_weekend"] = (out["dayofweek"] >= 5).astype(np.int64) # 主要解释np.int64，这个是numpy的形式，转化为bool值。 记得放在numpy库中，放了删掉这句话

    out["month_sin"] = np.sin(2 * np.pi * out["month"] / 12.0) # 下面的代码纯粹就是ai学习用的一些数学周期方面的东西，这里在下面也讲一下

    out["month_cos"] = np.cos(2 * np.pi * out["month"] / 12.0)

    out["dow_sin"] = np.sin(2 * np.pi * out["dayofweek"] / 7.0)

    out["dow_cos"] = np.cos(2 * np.pi * out["dayofweek"] / 7.0)

    out = out.drop(columns=["date", "id"], errors="ignore") # 删除data和id这两列，如果报错就忽略

    return out
```
### 最后周期的解释
为什么一个 `sin` 不够？

- `sin(θ) = sin(π-θ)`，会出现不同位置同一个值
- 也就是说只用 `sin` 时，模型可能分不清两个不同月份位置

直观例子（简化）：

- 某个角度和它的镜像角度，`sin` 值相同
- 但它们在一年中的位置不同

---

为什么加 `cos` 就好了？

- `sin` 和 `cos` 组成二维坐标 `(sinθ, cosθ)`
- 这个二维点能唯一表示圆上的角度位置（除极少数浮点误差）
- 相当于把“月份”放到一个圆上表示

---

为什么周期性特征有用？

因为时间变量本质是循环的：

- 12月和1月应该“接近”
- 周日和周一也应该“接近”

如果直接用整数：

- `month=12` 和 `month=1` 数值差是 11（看起来很远）
- 但实际在时间上相邻

用 `sin/cos` 后：

- 相邻周期点在特征空间也相邻
- 模型更容易学到季节性规律（销量常见）
### 第二版笔记
第二版
```python
def add_date_features(df: pd.DataFrame) -> pd.DataFrame:

out = df.copy()

# 复制一份数据，避免直接修改原始 df

dt = pd.to_datetime(out["date"], errors="coerce")

# 将 date 列转换为 datetime 类型

# errors="coerce" 表示异常日期会变成空值 NaT，而不是报错中断

out["year"] = dt.dt.year

# 提取年份

# 第一个 dt 是变量名；.dt 是 pandas 的日期访问器；.year 是取年份

out["month"] = dt.dt.month

# 提取月份（1~12）

out["day"] = dt.dt.day

# 提取日期（1~31）

out["dayofweek"] = dt.dt.dayofweek

# 提取星期几（0~6，通常 0=周一，6=周日）

out["weekofyear"] = dt.dt.isocalendar().week.astype(int)

# 提取一年中的第几周（ISO周）

# .isocalendar() 获取 ISO 日历信息（年/周/星期）

# .week 只取“周”

# .astype(int) 转成整数类型

out["quarter"] = dt.dt.quarter

# 提取季度（1~4）

out["dayofyear"] = dt.dt.dayofyear

# 提取一年中的第几天（1~365/366）

out["is_weekend"] = (out["dayofweek"] >= 5).astype(np.int64)

# 判断是否周末：周六/周日为 True，工作日为 False

# astype(np.int64) 把 True/False 转为 1/0（不是转成 bool，而是从 bool 转成整数）

out["month_sin"] = np.sin(2 * np.pi * out["month"] / 12.0)

out["month_cos"] = np.cos(2 * np.pi * out["month"] / 12.0)

# 把“月份”做成周期特征（sin/cos 成对使用）

# 目的：让模型理解 12月 和 1月 在时间上是相邻的

out["dow_sin"] = np.sin(2 * np.pi * out["dayofweek"] / 7.0)

out["dow_cos"] = np.cos(2 * np.pi * out["dayofweek"] / 7.0)

# 把“星期几”做成周期特征

# 目的：让模型理解 周日 和 周一 也是相邻的

out = out.drop(columns=["date", "id"], errors="ignore")

# 删除 date 和 id 两列

# date 已经被拆成多个时间特征

# id 通常只是编号，对预测帮助有限

# errors="ignore" 表示列不存在时忽略，不报错

return out
```


# Pandas和Numpy的转化

to_numpy()