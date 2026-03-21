==注意，有时候代码区并不会写import函数==
# 1.数组的创建

我们输入以下代码

```python
import numpy as np
arr_1 = np.array([1, 2, 3, 4, 5])  # 创建一个一维np数组
print(arr_1)  #[1 2 3 4 5]
print(type(arr_1))    # numpy数组的类型，即<numpy.ndarray>
```

这就创建了一个一维的np数组，对于二维的数组，我们可以这么创建

```python
import numpy as np
arr_2 = np.array([[1, 2, 3, 4, 5], [1, 2, 3, 4, 5]])   
print(arr_2)  #打印出的数组也会按照一个矩阵的形状打印
print(type(arr_2))    # numpy数组的类型  
print("数组形状", arr.shape)   # 数组形状，对于arr是（2，5），即二行五列
```

注意：
```python
arr = np.array([[1, 2, 3], [2, 3, 4], [3, 4, 5], [4, 5, 6]])
```
这也是一个二维数组，是四行三列的二维数组

我们也可以创建一个4\*4的随机数组，如下

```python
import numpy as np
np.random.seed(122) #固定随机种子，固定就是不随机，不固定就是随机
print(np.random.rand()) # 0 - 1中间的随机浮点数
arr = np.random.randint(0, 100, 16).reshape(4, 4) #0-100的随机整数，并按照4*4的矩阵呈现
```

扩展：
```python
import numpy as np

# 1. rand - [0,1) 均匀分布
print("rand:", np.random.rand(5))

# 2. randn - 标准正态分布（均值为0，标准差为1）
print("randn:", np.random.randn(5))

# 3. randint - 指定范围的随机整数
print("randint:", np.random.randint(0, 100, 5))

# 4. uniform - 指定范围的均匀分布
print("uniform:", np.random.uniform(0, 100, 5))

# 5. normal - 指定均值和标准差的正态分布
print("normal:", np.random.normal(50, 10, 5))
```

# 2.索引和切片

假设arr是一个一维数组

```python
import numpy as np
arr = np.array([1, 2, 3, 4, 5])
print(arr[0])  # 依旧从0开始索引
print(0:3)  #从0开始索引，取三个元素，即[0, 3)的切片
```

假设arr是一个二维数组

```python
arr = np.array([[1, 2, 3], [2, 3, 4], [3, 4, 5], [4, 5, 6]])
print(arr[0])  # 这时候就不是一个元素了，而是第0行，高了一个维度
print(arr[0:3])  # 也高了一个维度，即从第0行开始，切三行出来
print(arr[0][0]) # 这时候就是取的00这个元素  
```

# 3.运算

np数组的列表和python中的列表是不一样的，如下

```python
print([1, 2, 3] + [4, 5, 6])
print(np.array([1, 2, 3]) + np.array([4, 5, 6]))
print(np.array([1, 2, 3]) * np.array([4, 5, 6]))
```

结果是

```
[1, 2, 3, 4, 5, 6] 
[5 7 9]
[ 4 10 18]
```

可见，python中的列表是直接拼装起来的，但是np数组是满足常规数学运算的，加法是逐个元素相加，乘法是逐个元素相乘，即加减乘除都是对应位置元素进行运算(二维是什么样子的？二维与不同维度是什么样子的？)

```python
import numpy as np
np.random.seed(122) #固定随机种子，固定就是不随机，不固定就是随机
arr = np.random.randint(0, 100, 16).reshape(4, 4) #0-100的随机整数，并按照4*4的矩阵呈现
print(np.sum(arr[arr <= 10])) #对生成的随机的16个元素中的16个元素，抽取小于等于10的元素，求和
```

# 4.数组的形状运算（如转置）

- 数组的形状运算
```python
arr = np.array([[1, 2, 3], [2, 3, 4], [3, 4, 5], [4, 5, 6]])
print(arr.shape)
new_arr = arr.reshape(2, 6)
print(new_arr.shape)
print(new_arr)
```

结果是

```
(4, 3)
(2, 6)
[[1 2 3 2 3 4]
 [3 4 5 4 5 6]]
```

我们看到，前两行合并成了一行共六个元素，那如果按照下面的代码执行呢？

```python
arr = np.array([[1, 2, 3], [2, 3, 4], [3, 4, 5], [4, 5, 6]])
print(arr.shape)
new_arr = arr.reshape(2, 5)
print(new_arr.shape)
print(new_arr)
```

结果是

```
ValueError: cannot reshape array of size 12 into shape (2,5)
```

可见，这时候就会报错

- 数组的转置
```python
arr = np.array([[1, 2, 3], [2, 3, 4], [3, 4, 5], [4, 5, 6]])  
arr_T = arr.transpose() #因此，transpose就是转置的意思 
print(arr_T)
```

结果是

```
[[1 2 3 4]
 [2 3 4 5]
 [3 4 5 6]]
```


# 5.线性代数 统计 的应用

- 1.数组的点乘
```python
import numpy as np
arr_1 = np.array([1, 2, 3])  
arr_2 = np.array([4, 5, 6])  
arr_1_dot_arr_2 = np.dot(arr_1, arr_2)  
print(arr_1_dot_arr_2)
```

结果是

```
32
```

即通过 1\*4 + 2\*5 + 3\*6 运算得出的，同理，不同列数，不同行数的数组不能点乘

- 2.平均值的计算
```python
arr = np.array([[5, 2, 3], [2, 7, 4], [99, 4, 5], [4, 15, 6]])
print("数组的平均值", arr.mean())
print("数组的平均值", np.mean(arr)) # 这两种使用方式是等价的
```

- 其它功能
```python
print("数组的最大值", arr.max())
print("数组的最大值", np.max(arr)) # 99

print("数组的最小值", arr.min())
print("数组的最小值", np.min(arr)) # 2

print("数组的标准差", arr.std()) # 方差的平方根
print("数组的标准差", np.std(arr)) # 26.1374571576247

print("数组的排序", np.sort(arr)) # 这个是np的内部函数，只能这么写
#结果是数组的排序 [[ 2  3  5]
#[ 2  4  7]
#[ 4  5 99]
#[ 4  6 15]]

print("数组的排序", np.sort(arr.reshape(-1))) # 变为一维
# 数组的排序 [ 2  2  3  4  4  4  5  5  6  7 15 99]

print(arr > 10) #也是逐个比较
# [[False False False]
# [False False False]
# [ True False False]
# [False  True False]] 

print(arr[arr > 10]) #筛选大于10的元素
#[99 15]

print(arr[(arr > 10) & (arr < 5)]) #筛选   & 按位 与  等效于在每一位上进行一次 and
                                   #筛选   | 按位 或  等效于在每一位上进行一次 or 
```

# 6.数据的保存与导入

我们忽略上面的所有代码，从头开始
```python
import numpy as np
arr = np.array([[5, 2, 3], [2, 7, 4], [99, 4, 5], [4, 15, 6]])
np.save("arr1", arr)  #这个时候是会在文件目录下以 arr1.npy 的格式save一个numpy数组
```

这时候，我们把上面的代码全部注释掉，再写

```python
import numpy as np
np.load("arr.npy")
print(arr)
```

这时候就会重新导入arr进来





