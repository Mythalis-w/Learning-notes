```python
class BasicMLP(nn.Module):  
    def __init__(self, input_dim: int) -> None:  
        super().__init__()  # 继承nn.Module的初始化方法  
        # 搭建MLP网络结构：输入层→隐藏层→输出层，加入激活函数和Dropout防过拟合  
        self.net = nn.Sequential(  
            nn.Linear(input_dim, 512),  # 输入层→第一隐藏层（输入维度input_dim，输出512维）  
            nn.ReLU(),                  # 激活函数，引入非线性，帮助模型学习复杂关系  
            nn.Dropout(0.25),           # Dropout层，随机丢弃25%的神经元，防止过拟合  
            nn.Linear(512, 256),        # 第一隐藏层→第二隐藏层（512维→256维）  
            nn.ReLU(),                  # 激活函数  
            nn.Dropout(0.2),            # 随机丢弃20%的神经元  
            nn.Linear(256, 128),        # 第二隐藏层→第三隐藏层（256维→128维）  
            nn.ReLU(),                  # 激活函数  
            nn.Linear(128, 1)           # 输出层（128维→1维，对应销量预测值）  
        )  
  
    # 前向传播：定义模型的计算流程，输入x，输出预测结果(就是正着走一遍)  
    def forward(self, x: torch.Tensor) -> torch.Tensor:  
        # squeeze(1)：删除多余的维度（将[batch_size, 1]转为[batch_size]），适配后续损失计算  
        return self.net(x).squeeze(1)
```
