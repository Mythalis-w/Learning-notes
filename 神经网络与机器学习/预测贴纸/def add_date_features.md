
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

