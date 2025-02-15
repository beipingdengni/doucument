

参考学习文档：https://github.com/2951121599/ML

https://github.com/2951121599/ML/blob/master/%E6%9C%BA%E5%99%A8%E5%AD%A6%E4%B9%A0%E6%80%9D%E7%BB%B4%E5%AF%BC%E5%9B%BE.pdf





开始学习第一个：（博主：示木007）  https://blog.csdn.net/m0_58475958?type=blog

机器学习文章：（博主：showswoller）https://blog.csdn.net/jiebaoshayebuhui/category_12001149_3.html



如何评估机器学习模型的性能，除了交叉验证、MSE和RMSE外，还有哪些其他重要的指标？

评估机器学习模型的性能不仅限于交叉验证、均方误差（MSE）和均方根误差（RMSE），还有许多其他重要的指标。以下是一些常用的评估指标及其定义：

1. 准确率（Accuracy）：准确率是指模型正确预测的样本数量占总样本数量的比例。它适用于样本量大且类别均衡的情况。
2. 精确度（Precision）：精确度是指模型正确预测为正的样本数量占实际为正的样本数量的比例。它适用于样本量不均衡的情况，特别是在分类问题中。
3. 召回率（Recall）：召回率是指模型正确预测为正的样本数量占实际为正的样本数量的比例。它适用于样本量不均衡的情况，特别是在分类问题中。
4. F1分数（F1 Score）：F1分数是精确度和召回率的调和平均值，用于综合评估精确度和召回率。
5. ROC曲线：ROC曲线通过绘制不同阈值下的真正率（TPR）和假正率（FPR）来展示模型的性能。AUC（曲线下面积）则衡量了ROC曲线下的面积，AUC值越高，模型的性能越好。
6. Lift曲线：Lift曲线用于衡量模型在不同阈值下的相对提升效果，适用于需要高精度和高召回率的场景。
7. Gini系数：Gini系数是衡量分类器性能的一个指标，类似于AUC，但计算方法略有不同。Gini系数越高，模型的性能越好。
8. 信息熵：信息熵用于衡量模型的不确定性，信息熵越低，模型的预测越有确定性。
9. KL散度：KL散度用于衡量两个概率分布之间的差异，适用于评估模型预测与真实标签之间的差异。
10. 平均绝对误差（MAE）：MAE是指模型预测值与真实值之间差异的绝对值的平均值，适用于回归问题。
11. 混淆矩阵：混淆矩阵提供了一种可视化分类模型性能的方式，通过四个角度（真正率、假正率、假负率、漏报率）来展示模型的性能。
12. 决定系数（R²）：决定系数用于回归问题，表示模型解释的变异性比例，值越接近1，模型解释能力越强。

