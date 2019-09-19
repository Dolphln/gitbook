函数名：train\_test\_split  
所在包：**sklearn.model\_selection**  
功能：划分数据的训练集与测试集  
参数解读：train\_test\_split \(\*arrays，test\_size, train\_size, rondom\_state=None, shuffle=True, stratify=None\)

* arrays：特征数据和标签数据（array，list，dataframe等类型），要求所有数据长度相同。
* test\_size / train\_size: 测试集/训练集的大小，若输入小数表示比例，若输入整数表示数据个数。
* rondom\_state：随机种子（一个整数），其实就是一个划分标记，对于同一个数据集，如果rondom\_state相同，则划分结果也相同。
* shuffle：是否打乱数据的顺序，再划分，默认True。
* stratify：none或者array/series类型的数据，表示按这列进行分层采样。





举个栗子：

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



