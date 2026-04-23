# 1.张量

## 1.创建张量

从一些数据创建
```python
import torch

# 从列表创建
data = [[1, 2], [3, 4]] # 创建时后面可以加上dtype = torch.float32
x = torch.tensor(data)

# 从Numpy创建
import numpy as np
np_array = np.array(data) # 创建一个np数组
x_np = torch.from_numpy(np_array) #从np转为张量
```

除此以外，张量还能从内置函数去创建
```python
# 全0张量
zeros = torch.zeros(2, 3)
print(zeros)
# tensor([[0., 0., 0.],
#         [0., 0., 0.]])

# 全1张量
ones = torch.ones(2, 3)

# 随机张量（均匀分布 0~1）
rand = torch.rand(2, 3)

# 随机张量（标准正态分布）
randn = torch.randn(2, 3)

# 单位矩阵
eye = torch.eye(3)
# tensor([[1., 0., 0.],
#         [0., 1., 0.],
#         [0., 0., 1.]])

# 指定范围的等间隔
arange = torch.arange(0, 10, 2)  # start, end, step
# tensor([0, 2, 4, 6, 8])
```

以及创建指定类型的张量
```python
# 指定数据类型
x_int = torch.tensor([1, 2, 3], dtype=torch.int32)
x_float = torch.tensor([1, 2, 3], dtype=torch.float32)

# 从另一个张量创建（继承属性）
x_ones = torch.ones_like(x_int)  # 保持形状和类型
x_rand = torch.rand_like(x_int, dtype=torch.float)  # 覆盖类型
```

### 向量

向量就是一维的张量，可以通过索引来得到数据，也可以通过.numel() \[多维张量也可以用] 这个方法来得到有多少个数据

```python
x = torch.tensor([2.0, 0.0, 2.0, 1.0, -1.0])  # 一维张量
  
# 本题约定：要取的分量下标为 2（从 0 开始计数）  
# TODO: 取出该下标对应的分量  
elem = x[2]  
  
# TODO: 得到该向量的长度（元素个数）  
n = x.numel()  
  
print("elem:", elem)  
print("length:", n)
```

==注意：一维张量和行向量和列向量的区别==
```python
x_row = torch.tensor([[1.0, 2.0, 3.0]])  # shape: (1, 3)  行向量（1×n）
x_col = torch.tensor([[1.0], [2.0], [3.0]])  # shape: (3, 1)   列向量（n×1）
x_1d = torch.tensor([1.0, 2.0, 3.0])  # shape: (3,) 1维张量（n,） 在进行矩阵乘法的时候可以被视作行向量和列向量
```

## 2.张量的属性

```python
tensor = torch.rand(3, 4)

print(f"形状: {tensor.shape}")        # torch.Size([3, 4])
print(f"元素个数: {tensor.numel()}")   # 12
print(f"数据类型: {tensor.dtype}")     # torch.float32
print(f"设备: {tensor.device}")        # cpu
```

## 3.形状的变换

```python
tensor = torch.arange(12)  # tensor([0,1,2,3,4,5,6,7,8,9,10,11])

# reshape（改变形状）
reshaped = tensor.reshape(3, 4)
print(reshaped)
# tensor([[0, 1, 2, 3],
#         [4, 5, 6, 7],
#         [8, 9, 10, 11]])

# view（和 reshape 类似，但要求内存连续）
viewed = tensor.view(2, 6)

# 展平
flattened = reshaped.flatten()
# tensor([0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 11])

# 转置
transposed = reshaped.T
# tensor([[0, 4, 8],
#         [1, 5, 9],
#         [2, 6, 10],
#         [3, 7, 11]])

# 增加维度
unsqueezed = reshaped.unsqueeze(0)  # 在第0维增加一维
print(unsqueezed.shape)  # torch.Size([1, 3, 4])

# 删除维度为1的维度
squeezed = unsqueezed.squeeze()
print(squeezed.shape)  # torch.Size([3, 4])
```

## 4.算数运算和张量的广播

常见的算数运算
```python
a = torch.tensor([[1, 2], [3, 4]])
b = torch.tensor([[5, 6], [7, 8]])

# 逐元素运算
print(a + b)        # 加法: [[6,8],[10,12]]
print(a - b)        # 减法
print(a * b)        # 逐元素乘法
print(a / b)        # 除法
print(a ** 2)       # 平方

# 矩阵乘法
print(a @ b)        # 矩阵乘法（推荐）
print(a.matmul(b))  # 同上
print(torch.mm(a, b))  # 同上

# 标量运算
print(a + 10)       # 每个元素加10: [[11,12],[13,14]]
print(a * 2)        # 每个元素乘2: [[2,4],[6,8]]

# 聚合运算
print(a.sum())      # 所有元素求和: 10
print(a.mean())     # 平均值: 2.5
print(a.max())      # 最大值: 4
print(a.sum(dim=0)) # 按列求和: [4,6] 即对每列的所有元素求和,成为一行
print(a.sum(dim=1)) # 按行求和: [3,7] 即对每行的所有元素求和,然后把结果摆成一行

# 点积
v1 = torch.tensor([1, 2, 3])
v2 = torch.tensor([4, 5, 6])

# 点积：1*4 + 2*5 + 3*6 = 32
dot = torch.dot(v1, v2)
print(dot)  # tensor(32)

# 转置
a = torch.tensor([[1, 2, 3],
                  [4, 5, 6]])

print(a.T)
# tensor([[1, 4],
#         [2, 5],
#         [3, 6]])

# 范数(向量长度)
print(torch.norm(a))

# 指数
print(torch.exp(_tensor_))
```

### 广播
```python
import torch  
  
# 构造形状为 (3,1) 的 a
a = torch.tensor([[0],  
                  [1],  
                  [2]], dtype=torch.float32)  
  
# 构造形状为 (1,2) 的 b
b = torch.tensor([[0, 1]], dtype=torch.float32)  
  
C = a + b  
print(C)  
print("C.shape:", C.shape)

# tensor([[0, 1],
#        [1, 2],
#        [2, 3]]) 
```
如何判断能否广播：==看最后一维==
最后一维只有相同或者有一个为1的时候才可以广播
如：(1,2) (7,1)  可以广播运算，因为最后一维有一个1
`a` 形状 `(3,)`，`b` 形状 `(4, 3)` 最后一维都是3，可以广播

### 广播过程：

**a 广播**（从 `(3,1)` 到 `(3,2)`）：
```text
原始:    广播后:
[0]      [0, 0]
[1]  →   [1, 1]
[2]      [2, 2]
```

**b 广播**（从 `(1,2)` 到 `(3,2)`）：
```text
原始:         广播后:
[0, 1]  →    [0, 1]
             [0, 1]
             [0, 1]
```

## 5.索引与切片（局部赋值）

索引与切片的机制和Pandas库里写的一样,不再重复

### 局部赋值

```python
B = torch.arange(12).reshape(3, 4).float()  
  
# TODO: 把前两行赋值为 9  
B[:2,:] = 0
print(B)  
print("B.shape:", B.shape)
```

## 6.拼接 cat 与逻辑比较

### 拼接
