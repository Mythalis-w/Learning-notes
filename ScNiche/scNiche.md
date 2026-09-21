scNiche有M-GAE GFN MMIM
[课程补充--关于visium HD（Stereo-seq）scNiche与COMMOT的运用-腾讯云开发者社区-腾讯云](https://cloud.tencent.com/developer/article/2708778)

# 0. 原始数据
X的来历

在 Visium 数据里：
- 节点 = spot（每个 spot 是混合了多个细胞的小区域）
- N = spot 数量， D_i = 每个视图的特征维度
- 每个 X 都是一个 N × D_i 的矩阵

 X¹ = X_data — 细胞自身表达，来源（ cal_spatial_exp ）：

- 教程里 layer_key='X_scVI' → X_data 直接取预处理数据里已有的 scVI 潜在表示 （ N × D ，比如 50 维）
- 如果没指定 layer_key → 就是 adata.X （原始基因表达，稀疏转稠密，维度 = 基因数 33538）
- 可选： is_pca=True 时再用 PCA 降到 n_comps 即 X¹ = "每个 spot 自己（混合信号）的分子谱"。教程里指的是 scVI 压缩后的表示。
****
 X² = X_data_nbr — 邻居平均表达
这是最简单的数学加工（ L146-L168 ）：

1. 找邻居 ：用坐标 (x,y) 建 NearestNeighbors(n_neighbors=k+1) ，取每个 spot 最近的 k+1 个， 删掉自身 （ indices[0] ）
2. 取平均 ：对 spot i，把它的 k 个邻居的表达向量按列求均值
3. 所有 spot 拼成 X_data_nbr ，形状同 X¹（N × D）
即： X² = "我周围 k 个 spot 平均的分子谱" —— 是 X¹ 在空间上的"平滑/聚合版"。
****
 X³ = X_cn_norm — 邻居细胞类型组成
来源（ cal_spatial_neighbors ），分四步：

1. 坐标 ：取 spatial 取 x、y
2. 找物理邻居 ：同样 KNN（或 radius），得到每个 spot 的邻居索引
3. 统计邻居类型计数 ：把邻居的细胞类型标签做成 one-hot，求和
   - 得 X_cn ： N × C ， C =细胞类型数（如 75）
   - 行列含义：spot i 的邻居里，第 j 类细胞出现了几次
4. 按行归一化 ： X_cn_norm = X_cn / X_cn.sum(axis=1) ，得到比例
即： X³ = "我周围 k 个邻居里各种细胞类型各占多少比例" （T 5→0.25, B 3→0.15, ...）

 从 X 到 A 的关键一步：construct_graph
在 construct_graph 里，对 每个视图 （各自的 X¹、X²、X³）独立做：

```python
# 1. 在该视图特征空间上做 KNN（欧氏距离）
train_neighbors = NearestNeighbors(n_neighbors=knn+1).fit(data)
_, idx = train_neighbors.kneighbors(data)

# 2. 取对称邻接，转成 DGL 图，节点特征 = 该视图的 X
adj = train_neighbors.kneighbors_graph(data)
adj = adj + adj.T                      # 对称化
g = dgl.from_scipy(adj)
g.ndata['feat'] = data                      # X
g.ndata['adj']  = convert_adj(adj)          # A
```
所以 每个视图得到自己的一对 (X_i, A_i) ：

```
X¹ ─KNN(欧氏距离)→ A¹  （表达像不像）
X² ─KNN→ A²           （邻居环境像不
像）
X³ ─KNN→ A³           （细胞类型构成像
不像）
```


# 1. A的来历
拿到数据，adata（anndata）
- `adata.X`：细胞 × 基因表达矩阵
- `adata.obsm['spatial']`：每个细胞二维坐标 \((x,y)\)
- `adata.obs['cell_type']`：细胞类型注释


```
                    原始数据
                       │
          ┌────────────┼────────────┐
          ↓            ↓            ↓
       X^(1)         X^(2)        X^(3)
     自身表达       邻居平均表达    邻居细胞类型组成
          │            │            │
          ↓            ↓            ↓
        KNN          KNN          KNN
          │            │            │
          ↓            ↓            ↓
       A^(1)         A^(2)        A^(3)
          │            │            │
          └────────────┼────────────┘
                       ↓
                GFN进行图融合
                       ↓
                      A*
```
根据代码，代码会提前设定类别顺序，然后后面的矩阵严格按照类别顺序进行


$X^1$：细胞自身表达，如$X^1$就是`adata.obsm['X_data'] = data_raw` 数据实验测得

取出坐标矩阵 $(X^1 \in \mathbb R^{N×D})$  ，如有N个细胞D个特征
$$ \begin{bmatrix} 1.0 & 2.0 \\ 3.0 & 4.0 \\ 5.0 & 6.0 \end{bmatrix} $$
如这里就是3个细胞两个特征，每一行都是细胞自己的基因表达的特征
然后把$X^1$扔进KNN中得到$A^1$ 

****
$X^2$：空间邻居平均表达
首先我们要设定取多少个邻居，如果取3，然后得到3个邻居是`[5,8,10]`，
```
cell 5  = [10, 20, 30, 40]
cell 8  = [20, 10, 20, 30]
cell 12 = [30, 30, 10, 20]
```
然后再按列取平均（每一行都类似$X^1$，是自己的基因表达）

那$X^2$就是`[20 20 20 30]`
然后再把所有细胞的邻居平均表达拼成一个矩阵
扔进KNN中得到$A^2$
****
$X^3$：（一个spot内）空间邻居细胞类型组成
通过得知每个细胞周围是取的邻居是什么类型（这里假设邻居取20个）
```
T cell        5
B cell        3
Macrophage    8
NK cell       4
```
再归一化
```
T cell        0.25
B cell        0.15
Macrophage    0.40
NK cell       0.20
```
$X^3$=`[0.25, 0.15, 0.40, 0.20]`
然后再把所有空间邻居细胞类型组成拼成一个矩阵
扔进KNN中得到$A^3$
***
三个X代表三个特征空间，我们通过KNN分别算每个X的欧式距离，从而得到哪两个细胞更相似，然后会选出knn=3个（假设）
```
Cell1
 ├── Cell7
 ├── Cell3
 └── Cell12
```

然后会生成一个邻接矩阵（这个仅作为举例子，实际上，我们仅能确定第一行，因为2号不知道和谁邻接，所以每一行理论上是有knn个1）
```
        C1  C2  C3  C4 C5 C6  C7  …… C12
C1       0   0   1   0  0  0   1  ……   1
C2       1   0   1   0
C3       1   1   0   0
C4       1   0   1   0
```


$A^2 A^3$同理，

X是
```
    T/gene1     B/gene2     NK/gene3
C1    0.2          0.7          0.1
C2
C3
```
这种的


拿到三个A，做对称化，$(\boldsymbol A_{\text{sym}} = \boldsymbol A \lor \boldsymbol A^\mathrm T)$ （i的邻居是j，但是j的邻居不一定是i，可能j有比i更近的）对称化后的A才是GFN中用到的A。 
```python
adj = train_neighbors.kneighbors_graph(data)
adj = adj + adj.T.multiply(adj.T > adj) - adj.multiply(adj.T > adj)
```


# 1.1 KNN
首先是计算$x_0$与X中每个向量的距离（欧式距离），组成dist列表
然后取出dist前k个小的元素，说明$x_0$与这K个元素对应的向量距离最近

然后会保存每个细胞的邻居的索引列表
生成一个NN的矩阵，全部置零
然后逐个细胞遍历索引列表，在NN的矩阵中赋值为1（这也是后面A需要自环的原因，对角线全是0）



# 2.GFN

两个全连接层，M-GAE于GFN训练时信息共享？，激活函数是ReLU，初始输入$G_0$，损失函数是MSE
最终目的是出来$A^* = G_2$

首先，$G_0$，是三个A的加和，每个元素 $(G_0[i,j])$ 取值：0,1,2,3，代表 i‑j 在多少个 view 图中存在边。

G0扔进GFN中，第一次用ReLU函数，第二次直接输出 A星
损失函数用MSE，把每一层的A和$A^*$的损失累加
```python
class GFN(nn.Module):
    def __init__(self, input_size, hidden_size):
        super(GFN, self).__init__()
        self.gfn = nn.Sequential(
            nn.Linear(input_size, hidden_size),
            nn.ReLU(),
            nn.Linear(hidden_size, input_size)
        )

    def forward(self, x):
        out = self.gfn(x)
        return out

```

# 3.M-GAE
第一步对每个视图都做GCN，(三个A都加上自环，因为邻接矩阵对角线是0)其中$\tilde A^{(v)}$ 是自环后的A

$Z_{(1)}^{(v)}=\delta\left( \left(\tilde D^{(v)}\right)^{-\frac12} \tilde A^{(v)} \left(\tilde D^{(v)}\right)^{-\frac12} X^{(v)} W_{(1)}^{(v)} \right) \tag1$
拿到三个Z1，代表三个view

第二步对这三个Z分别再做一次GCN（论文），然后通过W学习每个视图的重要程度做融合得到一个$Z_2$
$Z_{(2)}=\delta\left( \sum_{v=1}^V W_a^{(v)} \left( (\tilde D^{(v)})^{-\frac12}\tilde A^{(v)}(\tilde D^{(v)})^{-\frac12} Z_{(1)}^{(v)} W_{(2)}^{(v)} \right) \right) \tag2$

通过GFN得到的$A^*$ ，和Z2再做一个GCN，得到最后的Z（N×D）（cell数量：表达特征）
$Z=\delta\left( \left(\tilde D^*\right)^{-\frac12} \tilde A^* \left(\tilde D^*\right)^{-\frac12} Z_{(2)} W_{(3)} \right) \tag3$

如果是训练的时候，还要跑公式4、5
$\hat A^{(v)}=\text{sigmoid}\big(Z\cdot W^{(v)}\cdot Z^T\big) \tag4$
$L_{rec}= \sum_{v=1}^V L_{rec}^{(v)}=\sum_{v=1}^V loss\big(A^{(v)},\hat A^{(v)}\big) \tag5$
最后我们的目的就是让公式5这两个逼近，让损失率减少

问题：
```python
def forward(self, graphs, data, device):  
    feats = [self._apply_gcn_layers(layer, graphs[i], data[i]) for i, layer in enumerate(self.layers_per_view)]  
    feat_fusion = self.featfusion(*feats)  
    adj_r, g = self.consensus_graph(graphs, device)  
  
    for conv in self.layer_m:  
        feat_fusion = conv(dgl.add_self_loop(g), feat_fusion)  
  
    adj_rec = {i: self.decoder(feat_fusion) for i in range(self.views)}  
  
    return adj_r, adj_rec, feat_fusion
```
这个是Github的代码，前向传播的过程中，


GCN只进行一个维度D的缩小，比如一开始的表达特征是3000维，但是最后就剩下50维了，方便Kmeans进行聚类（GCN的优势就是在降维的时候还能同时考虑周围的邻居关系降维，不会丢失这个关系）
# 4.MMIM

预处理得到mik

举例：

设定条件：
- 细胞总数 \(N=3\)（细胞 0、细胞 1、细胞 2）
- 隐表征维度 \(d=2\)，每个细胞输出 2 维向量
- 每个细胞取 **1 个空间邻居**，所以 `mik` 形状：$\boldsymbol{[3,1]}$

根据KNN：
- 细胞 0 的空间邻居：细胞 1
- 细胞 1 的空间邻居：细胞 0
- 细胞 2 的空间邻居：细胞 1、
$\text{mik}= \begin{bmatrix} 1 \\ 0 \\ 1 \end{bmatrix}$

假设M-GAE输出的Z是

$Z= \begin{bmatrix} z_0 \\ z_1 \\ z_2 \end{bmatrix}= \begin{bmatrix} 0.1 & 0.2 \\ &\text{细胞0}\\ 0.3 & 0.4 \\ &\text{细胞1}\\ 0.5 & 0.6 \end{bmatrix}\quad \text{细胞2}$

根据mik，取出邻居的表征，mik作为下标
$Z_{pos}= \begin{bmatrix} 0.3 & 0.4 \\ 0.1 & 0.2 \\ 0.3 & 0.4 \end{bmatrix}$


$Z_{positive}= \left[ \begin{array}{cc|cc} 0.1 & 0.2 & 0.3 & 0.4 \\ 0.3 & 0.4 & 0.1 & 0.2 \\ 0.5 & 0.6 & 0.3 & 0.4 \end{array} \right]$
然后拼接得到正样本矩阵
然后送进判别器（判别器对第i组**真实邻居对**给出 0~1 的概率），输出 `z_scores` `shape (3,1)`每个值 0~1：代表判别器认为 “这一组是不是真实空间邻居对” 的概率。

然后同样是负样本$Z_{shuf}=Z[[2,0,1]]= \begin{bmatrix} 0.5 & 0.6 \\ 0.1 & 0.2 \\ 0.3 & 0.4 \end{bmatrix}$ 这个矩阵是随机打乱的
然后和Z进行拼接得到负样本矩阵
送入判别器，希望判别器输出得分**全部趋近于 0**
$L_{mim}=-\frac1N\sum_{i=1}^N\Big[\log D(z_i,z_{pos,i}) \;+\; \log\big(1-D(z_i,z_{shuf,i})\big)\Big]$
最小化$L_{mim}$，也就是最大化互信息。

最大化互信息 $I(Z,Z')$，Z是细胞表征，\(Z'\)是它邻居的表征；
```python
delta = 1e-7
loss_pos = torch.log(z_scores + delta)
loss_neg = torch.log(1.0 - z_shuf_scores + delta)
L_mim = - torch.mean( loss_pos + loss_neg )
```


![[ScNiche.png]]


# 最后
$L_{total}=L_{rec}+L_{gre}+\boldsymbol{L_{mim}}$
rec有一个超参数a，mim有一个超参数b，默认全是1
```
loss_total = loss_rec + loss_gre + loss_mim
loss_total.backward()
```
反向传播后，梯度流向三处：
1. **M‑GAE 编码器（最重要）**：更新权重，让空间邻居的z变得更相似
2. GFN 网络
3. `model_d`判别器网络：提升判别正负样本对的能力



# Kmeans 和 FMI

scNiche 通过 M-GAE 学习细胞的低维嵌入表示\(\boldsymbol Z\)，将生物学特征相似的细胞映射到向量空间中相近位置；随后在嵌入空间上使用 K-Means 算法，依据向量距离对细胞进行无监督聚类，识别细胞生态位。

先设定簇数量 K，算法自动把样本分成 K 组，对于K的选择，要用到FMI，这里先将K-means的算法，

在Z这个D维向量空间中，一共有N个点（cell），每个点都是一个D维度点，我们用FMI选择好K后，我们会决定把这个N个点分成K类

对于K类，我们随机选择K个点（标号为1、2、3……、K）

1. 然后计算剩下的点距离这K个点中的欧式距离，假设有一个点距离点i（i是K个点其中的一个）最近，然后把最近的那个标记为i，循环，标记完所有的点

2. 然后我们把所有标记为1的点的坐标平均，作为新的1号点，其余的K-1个点同理

再次循环上面的两步，直到我们设置的步数被用完，或者这K个点的坐标不再移动，那么算法结束，所有的点被分为了K类

K-means++/n_init?

接下来是FMI，可以用来挑选用哪一个K最好（推荐固定种子，不然每次FMI后得到的K可能不一样）

执行FMI，我们要先给一个K的范围，比如`K_tmp=[2,3,4,5,6,7,8]`，代表K可能在这个7类之中，然后遍历这个列表，在同一个Z中，跑三次K-means，分别是K为K-1、K、K+1的情况，然后计算FMI（K-1,K）和FMI(K,K+1)，然后avgFMI，作为当前K的稳定性得分，最后遍历完毕后选择avgFMI最高的那一个作为K的最优值

FMI的计算
![[FMI.png]]

结束。
