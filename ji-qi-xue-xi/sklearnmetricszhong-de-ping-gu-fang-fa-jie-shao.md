# **accuracy\_score**

分类准确率分数是指所有分类正确的百分比。分类准确率这一衡量分类器的标准比较容易理解，但是它不能告诉你响应值的潜在分布，并且它也不能告诉你分类器犯错的类型。

**形式**

```
sklearn.metrics.accuracy_score(y_true, y_pred, normalize=True, sample_weight=None)
```

**normalize**：默认值为True，返回正确分类的比例；如果为False，返回正确分类的样本数

* 示例

```
>>>import numpy as np
 
>>>from sklearn.metrics import accuracy_score
 
>>>y_pred = [0, 2, 1, 3]
 
>>>y_true = [0, 1, 2, 3]
 
>>>accuracy_score(y_true, y_pred)
 
0.5
 
>>>accuracy_score(y_true, y_pred, normalize=False)
 
2
```

# **recall\_score**

召回率 =提取出的正确信息条数 /样本中的信息条数。通俗地说，**就是所有准确的条目有多少被检索出来了**。

**形式**

```
sklearn.metrics.recall_score(y_true, y_pred, labels=None, pos_label=1,average='binary', sample_weight=None)
```



