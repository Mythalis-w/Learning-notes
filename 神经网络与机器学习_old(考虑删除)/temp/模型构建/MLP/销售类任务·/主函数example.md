```python
def main() -> None:  
    # 读取训练集、测试集（csv格式）  
    train_df = pd.read_csv("train.csv")  
    test_df = pd.read_csv("test.csv")  
    # 删除训练集中销量为空的行（避免缺失目标值，影响训练）  
    train_df = train_df.dropna(subset=["num_sold"]).copy()  
	# .drop(...) → 删列 / 删行根据标签删）
	# .dropna(...) → 删空值所在的行（根据有没有 NaN 删）
  
    # 构建特征：获取处理好的训练特征、目标值、测试特征、训练集日期  
    x_train, y, x_test, train_dates = build_features(train_df, test_df)  
    # 对目标值（销量）做log1p变换（解决销量分布偏态问题，让模型更易学习）  
    y_log = np.log1p(y).astype(np.float32)  
  
    # 多种子训练：用3个不同种子训练，取预测结果平均（提升模型泛化能力，降低随机误差）  
    seeds = [42, 2024, 3407]  # 3个随机种子（可修改）  
    pred_logs = []  # 保存每个种子的测试集对数预测值  
    val_scores = []  # 保存每个种子的最优验证RMSLE  
    for seed in seeds:  
        # 单种子训练，获取预测结果和最优验证分数  
        pred_log, best_rmsle = train_one_seed(seed, x_train, y_log, train_dates, x_test)  
        pred_logs.append(pred_log)  # 加入预测结果列表  
        val_scores.append(best_rmsle)  # 加入验证分数列表  
  
    # 多种子融合：对3个种子的预测结果取平均（降低随机波动，提升预测稳定性）  
    pred_log = np.mean(np.stack(pred_logs, axis=0), axis=0)  
    # 对数预测值还原为原始销量（log1p的逆操作是expm1）  
    pred = np.expm1(pred_log)  
    # 裁剪预测结果：销量不能为负，将小于0的值设为0  
    pred = np.clip(pred, 0, None)  
  
    # 生成提交文件：按比赛要求格式，id对应测试集id，num_sold对应预测销量  
    submission = pd.DataFrame({"id": test_df["id"], "num_sold": pred})  
    submission["num_sold"] = submission["num_sold"].round(3).astype(np.float32)  # 保留3位小数，转float32  
    submission.to_csv("submission_basic_mlp.csv", index=False)  # 保存为csv，不保留行索引  
  
    # 打印训练结果（方便查看模型效果）  
    print(f"Seeds: {seeds}")  
    print(f"Best Val RMSLE per seed: {[round(s, 6) for s in val_scores]}")  
    print(f"Mean Val RMSLE: {np.mean(val_scores):.6f}")  
    print("预测完成，已保存: submission_basic_mlp.csv")  
    print(submission.head(10).to_string(index=False))  # 打印前10条预测结果
```

这里我们对上面所有的铺垫写一个主函数，我们只对上面每出现过的代码做一下讲解\

首先是`y_log = np.log1p(y).astype(np.float32)` ，我们要对y进行一个log化，就是ln，这也是前面为什么有y_log的原因，如果不用log，大数字会占据主导,模型会拼命去拟合大销量，小销量全错。用了log会减少这种情况

`pred_log = np.mean(np.stack(pred_logs, axis=0), axis=0) `
这个是把3个种子的预测值按列堆叠，然后按行累加，再取平均，提升泛化能力

```text
[
  [种子1的预测1, 种子1的预测2, ... 种子1的预测10000],
  [种子2的预测1, 种子2的预测2, ... 种子2的预测10000],
  [种子3的预测1, 种子3的预测2, ... 种子3的预测10000],
]

变为

最终预测1 = (种子1预测1 + 种子2预测1 + 种子3预测1) / 3
最终预测2 = (种子1预测2 + 种子2预测2 + 种子3预测2) / 3
...

```
	

```python
    # 对数预测值还原为原始销量（log1p的逆操作是expm1）  
    pred = np.expm1(pred_log)  
    # 裁剪预测结果：销量不能为负，将小于0的值设为0  
    pred = np.clip(pred, 0, None) 
```
最后两个注释已经说的很明白了，不在赘述