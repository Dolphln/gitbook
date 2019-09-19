函数名：train\_test\_split  
所在包：**sklearn.model\_selection**  
功能：划分数据的训练集与测试集

使用形式：

```py
from sklearn.model_selection import train_test_split 

X_train, X_test, y_train, y_test = train_test_split(train_data,train_target,test_size=0.2, random_state=0)
```

参数解读：

* train\_data：样本特征集

* train\_target：样本的标签集

* test\_size / train\_size: 测试集/训练集的大小，若输入小数表示比例，若输入整数表示数据个数。

* rondom\_state：随机种子（一个整数），其实就是一个划分标记，对于同一个数据集，如果rondom\_state相同，则划分结果也相同。

* shuffle：是否打乱数据的顺序，再划分，默认True。
* stratify：none或者array/series类型的数据，表示按这列进行分层采样。
* X\_train,y\_train:构成了训练集

* X\_test,y\_test：构成了测试集

* 举个栗子：

```py
特征数据：data
   a  b  c
0  1  2  3
1  1  3  6
2  2  3  8
3  1  5  7
4  2  4  8
5  2  3  6
6  1  4  8
7  2  3  6
标签数据：label
[2,3,5,6,8,0,2,3]

#划分
xtrain,xtest,ytrain,ytest=train_test_split(data,label,test_size=0.2,stratify=data['a'],random_state=1)
训练特征集：
   a  b  c
0  1  2  3
2  2  3  8
3  1  5  7
5  2  3  6
6  1  4  8
4  2  4  8
测试特征集：
   a  b  c
1  1  3  6
7  2  3  6

训练集与测试集按照a列来分层采样，且无论重复多少次上述语句，划分结果都相同。
```

举例：

```py
>>> import numpy as np
>>> from sklearn.model_selection import train_test_split
>>> x = np.arange(15).reshape(-1, 3)  #生成5行3列的一个矩阵
>>> x
array([[ 0,  1,  2],
       [ 3,  4,  5],
       [ 6,  7,  8],
       [ 9, 10, 11],
       [12, 13, 14]])
>>> y = np.arange(5)  #5个数的向量
>>> y
array([0, 1, 2, 3, 4])
>>> x_train, x_test, y_train, y_test = train_test_split(x, y, test_size=0.2, random_state=1)
>>> x_train
array([[ 3,  4,  5],
       [12, 13, 14],
       [ 0,  1,  2],
       [ 9, 10, 11]])
>>> x_test
array([[6, 7, 8]])
>>> y_train
array([1, 4, 0, 3])
>>> y_test
array([2])
>>>
```

说明

```
x，y是原始的数据集。x_train,y_train 是原始数据集划分出来作为训练模型的，fit模型的时候用。
x_test,y_test 这部分的数据不参与模型的训练，而是用于评价训练出来的模型好坏，score评分的时候用。
test_size=0.2 测试集的划分比例
random_state=1 随机种子，如果随机种子一样，则随机生成的数据集是相同的
```

使用KNN

```
from sklearn.neighbors import KNeighborsClassifier
knn_clf = KNeighborsClassifier()
knn_clf.fit(x_train, y_train) #用fit训练模型，x_train, y_train是第一步划分的数据集。
knn_clf.score(x_test, y_test) #score测试模型，x_test, y_test是第一步划分得到的
```



