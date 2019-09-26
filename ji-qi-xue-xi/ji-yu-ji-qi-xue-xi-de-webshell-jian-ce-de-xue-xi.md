本篇文章仅探讨PHPwebshell的检测

参考文章：

[兜哥：基于机器学习的 Webshell 发现技术探索](https://mp.weixin.qq.com/s?__biz=MzIwNjEwNTQ4Mw==&mid=2651577090&idx=1&sn=924b14ba842f57c34f06995416a98360&chksm=8cd9c5e6bbae4cf0e3eed6192133c6c87de47cfcc911fca90d86f1383d5ec2f6f1cf661aaeb6&mpshare=1&scene=1&srcid=0118yl2ryPVxJto00p3uvrhy#rd)

[云众可信：原创干货 \| 基于机器学习的webshell检测踩坑小记](https://mp.weixin.qq.com/s/OPzf4VRYZBc6YL6QIrJuHQ)

[初探机器学习检测 PHP Webshell](https://www.cnblogs.com/wenr0/p/9559207.html)

参考源码：

[https://github.com/hi-WenR0/MLCheckWebshell/blob/master/train.py](https://github.com/hi-WenR0/MLCheckWebshell/blob/master/train.py)

### 什么是webshell？

Webshell是一种基于web应用的后门程序，是黑客通过服务器漏洞或其他方式提权后，为了维持权限所部署的权限木马。Webshell的危害非常大，通常为网站权限，可以对网站内容进行随意修改、删除，甚至可以进一步利用系统漏洞对服务器提权，拿到服务器权限。

### 解决方法：

webshell检测方法较多，有静态检测、动态检测、日志检测、语法检测、统计学检测等等。本文重点讨论webshell的静态检测方法，静态检测也是最主要的检测手段。

静态检测通过匹配特征码、特征值、危险函数函数来查找webshell的方法，只能查找已知的webshell，并且误报率漏报率会比较高，但是如果规则完善，可以减低误报率，但是漏报率必定会有所提高。优点是快速方便，对已知的webshell查找准确率高，部署方便，一个脚本就能搞定。缺点漏报率、误报率高，无法查找0day型webshell，而且容易被绕过。

机器学习最大的优点是它具有泛化能力，也就是可以举一反三。基于机器学习的webshell检测可以有效的提升准确率和减低漏报率。本文使用的算法为兜哥的opcode+tfidf算法。如下是常规检测和机器学习检测率的比较。

![](/assets/shell-11.png)

### 

### **0x01 机器学习衡量指标**

#### 1、混淆矩阵

混淆矩阵（Confusion Matrix）又被称为错误矩阵，通过它可以直观地观察到算法的效果。在二分类问题中，可以用一个2×2的矩阵表示，如下表所示。其中，FP为表示实际为假预测为真，也就是误报，FN表示实际为真预测为假，也就是漏报。

![](/assets/shell-2.png)

#### 2、准确率和召回率

![](/assets/shell-3.png)

机器学习中最常用的指标就是准确率和召回率。准确率也叫查准率，要提高查准率就要降低误报。召回率也叫查全率，要提高查全率就要降低漏报。

#### 3、F1-Score

![](/assets/shell-4.png)

人们通常使用准确率和召回率这两个指标，来评价二分类模型的分析效果。但是当这两个指标发生冲突时，我们很难在模型之间进行比较。此时可以使用F1-score来进行综合评判。

### **0x02 数据集**

所用数据集均来源于github，需要两类数据，黑名单和白名单。白名单可以从以下链接获取。

```
https://github.com/topics/php?o=desc&s=stars
```

黑名单使用以下项目：

```
https://github.com/tennc/webshell
https://github.com/ysrc/webshell-sample
https://github.com/tanjiti/webshellSample
```

由于github下载速度较慢，以下是github中star排名前350的php项目的白名单和12个webshell项目的黑名单的备份。测试算法时可以只需取部分项目即可。

```
链接: https://pan.baidu.com/s/12FWAN-jIWTqjOlwB5V0DuQ 提取码: 31bi
```

### **0x03 特征提取**

#### 1.词袋和TF-IDF模型

词袋模型是指将句子或文本中的每个单词看成一个集合，并统计其出现的次数。词袋模型被广泛应用在文件分类，词出现的频率可以用来当作训练分类器的特征。对一篇文章进行特征化，最常见的方式就是词袋。

TF-IDF（词频-逆文本频率指数）模型是一种用以评估一个词对于一个文件集或一个语料库中的其中一份文件的重要程度。一个字词的重要性随着它在文件中出现的次数成正比增加，同时会随着它在语料库中出现的频率成反比下降。

函数`get_feature_by_tfidf`的作用是传文本数组进去，可以返回一个特征矩阵，原理就是使用词袋和TF-IDF模型计算文本特征。

```py
def get_feature_by_tfidf(x, max_features=None):
    cv = CountVectorizer(ngram_range=(3, 3), decode_error="ignore",
                         max_features=max_features,token_pattern=r'\b\w+\b', 
                         min_df=1, max_df=1.0)
    x = cv.fit_transform(x).toarray()
    transformer = TfidfTransformer(smooth_idf=False)
    transformer = transformer.fit_transform(x)
    x = transformer.toarray()
    return x
```

#### 

#### 2、opcode模型

opcode是计算机指令的一部分，也叫字节码，一个php文件可以抽取出一个指令序列，如ADD、ECHO、RETURN。由于直接对php文件使用词袋和TF-IDF进行模型训练会消耗大量计算资源，使用opcode模型进行降维可以有效提升模型效率和模型的准确率。由于opcode只关心操作指令，不关心函数名、定义等，因此可以有效的检测一些加密、混淆的代码。

要使用opcode模型，需要装一个vld的扩展，需要注意的是不同版本的vld支持不同版本的PHP，如vld 0.11.1版本支持PHP 5.4，vld 0.13.0支持PHP 5.6。下载链接如下：

```
Windows：http://pecl.php.net/package/vld/0.14.0/windows

Linux：http://pecl.php.net/package/vld
```

Windows下载的是dll文件，只需将其中的php\_vld.dll放到PHP安装目录/ext目录下,编辑php.ini文件添加extension=php\_vld.dll即可。

Linux的安装命令如下，安装完后配置php.ini,将extension=vld.so添加进去。

```
tar zxvf vld-0.xx.x.tgz
cd vld-0.xx.x
phpize
./configure
make && make install
```

安装完成后，使用php -dvld.active=1 -dvld.execute=0 1.php即可获取1.php文件的opcode。对于&lt;?php echo "test"; ?&gt;的opcode序列为ECHO RETURN,如下图所示：

![](/assets/shell-5.png)

对于一句话&lt;?php @eval\($\_POST\['a'\]\);?&gt;的opcode序列为BEGIN\_SILENCE FETCH\_R FETCH\_DIM\_R INCLUDE\_OR\_EVAL END\_SILENCE RETURN,如下图所示：

![](/assets/shell-6.png)

函数`load_php_opcode`的作用是提取一个php文件的opcode序列。

```py
def load_php_opcode(php_filename):
    try:
        output = subprocess.check_output(['php.exe', '-dvld.active=1',
                                          '-dvld.execute=0', php_filename],
                                         stderr=subprocess.STDOUT)
        tokens = re.findall(r'\s(\b[A-Z_]+\b)\s', output)
        t = " ".join(tokens)
        return t
    except:
        return " "
```

由于提取opcode会消耗较多的时间，函数`load_php_opcode_from_dir_with_file`会提取一个文件夹中所有php文件的opcode到指定的文件中，作为训练和预测时的特征文件。

```py
def load_php_opcode_from_dir_with_file(dir, file_name):
    print "load php opcode from dir => " + dir
    for root, dirs, files in os.walk(dir):
        for filename in files:
            if filename.endswith('.php'):
                try:
                    full_path = os.path.join(root, filename)
                    file_content = load_php_opcode(full_path)
                    with open(file_name, "a+") as f:
                        f.write(file_content + "\n")
                except:
                    continue
```

依次读取保存 WebShell 样本以及正常PHP文件的目录，加载对应的opcode字符串，其中标记 WebShell 为1，正常PHP文件为0。

```py
white_file_list = []
black_file_list = []

f1=open("./opcode_white.txt","r")
f2=open("./opcode_black.txt","r")

with open("./opcode_white.txt","r") as f:
        for line in f:
                white_file_list.append(line.strip("\n"))

with open("./opcode_black.txt","r") as f:
        for line in f:
                black_file_list.append(line.strip("\n"))

len_white_file_list=len(white_file_list)
len_black_file_list=len(black_file_list)

x = white_file_list+black_file_list
y = [0]*len_white_file_list + [1]*len_black_file_list
```

### 

### **0x04 模型训练及检测**

使用2-gram处理opcode字符串,其中通过设置`ngram_range=(2, 2)`就可以达到使用2-gram的目的，同理如果使用3-gram设置`ngram_range=(3, 3)`即可。

```py
CV = CountVectorizer(ngram_range=(2, 2), decode_error="ignore",max_features=max_features,
token_pattern = r'\b\w+\b',min_df=1, max_df=1.0)
x = CV.fit_transform(x).toarray()
```

使用TF-IDF进一步处理。（我觉得是因为单一的使用词袋模型只是将文档转化为数值矩阵，不能体现词语的重要性，继续使用tf-idf进行分析会更准确一些）

```py
transformer = TfidfTransformer(smooth_idf=False)
x_tfidf = transformer.fit_transform(x)
x = x_tfidf.toarray()
```

CountVectorizer 的作用是把一些列文档的集合转化成数值矩阵。

TfidfTransformer 的作用是把数值矩阵规范化为 tf 或 tf-idf 。

代码介绍：

首先，我们用了刚才写的prepare\_data\(\)函数来获取我们的数据集。然后，创建了一个CountVectorizer 对象，初始化的过程中，我们告诉CountVectorizer对象，ngram的上下限为\(2,2\) 【ngram\_range=\(3,3\)】，当出现解码错误的时候，直接忽略【decode\_error=”ignore”】，匹配token的方式是【r”\b\w+\b”】，这样匹配我们之前用空格来隔离每个opcode 的值。

然后我们用 cv.fit\_transform\(X\).toarray\(\) 来“格式化”我们的结果，最终是一个矩阵。

接着创建一个TfidfTransformer对象，用同样的方式处理一次我们刚才得到的总数据值。

#### 1、朴素贝叶斯算法

使用朴素贝叶斯算法，特征提取使用词袋&TF-IDF模型，完整的处理流程为：

1. 将 WebShell 样本以及常见 PHP 开源软件的文件提取词袋。

2. 使用 TF-IDF 处理。

3. 随机划分为训练集和测试集。

4. 使用朴素贝叶斯算法在训练集上训练，获得模型数据。

5. 使用模型数据在测试集上进行预测。

6. 验证朴素贝叶斯算法预测效果。

![](/assets/shell-7.png)

```py
def do_gnb(x, y):
    x_train, x_test, y_train, y_test = train_test_split(x, y, test_size=0.4, 
                                                        random_state=0)
    clf = GaussianNB()
    clf.fit(x_train, y_train)
    joblib.dump(clf, 'model/gnb.pkl')
    y_pred = clf.predict(x_test)
    do_metrics(y_test, y_pred)
```

代码介绍：

使用train\_test\_split函数来获取**打乱的随机的**测试集和训练集。这时候，黑名单中的文件和白名单中的文件排列顺序就被随机打乱了，但是X\[i\] 和 y\[i\] 的对应关系没有改变，训练集和测试集在总数聚集中分别占比60%和40%。

接下来，创建一个GaussianNB 对象，在Scikit-learn中，已经内置好的算法对象可以直接进行训练，输入内容为训练集的数据（X\_train） 和 训练集的标签（y\_train）。

执行完上面这个语句以后，我们就会得到一个已经训练完成的gnb训练对象，我们用测试集\(X\_test\) 去预测得到我们的y\_pred 值（预测出来的类型）。

然后我们对比原本的 y\_test 和 用训练算法得到的结果 y\_pred。

```
print(metrics.accuracy_score(y_test, y_pred))
print(metrics.confusion_matrix(y_test, y_pred))
print(metrics.precision_score(y_test, y_pred))
print(metrics.recall_score(y_test, y_pred))
```

### ![](/assets/shell-8.png)

算法结果不理想

2、随机森林算法

```py
clf = RandomForestClassifier(n_estimators=50)
clf.fit(x_train, y_train)
joblib.dump(clf, 'pkl/rf.pkl')
y_pred = clf.predict(x_test)

print(metrics.accuracy_score(y_test, y_pred))
print(metrics.confusion_matrix(y_test, y_pred))
print(metrics.precision_score(y_test, y_pred))
print(metrics.recall_score(y_test, y_pred))
```

![](/assets/shell-9.png)

效果比

### **0x05 预测新样本**

#### 1.预测单个文件

需要注意的是，由于采用TF-IDF模型进行特征提取，在预测新样本时，要加载训练时的黑白样本特征作为语料库，进而计算出新样本的特征矩阵。加载语料库的代码如下：

```py
def get_old_data():
    white_file_list = []
    black_file_list = []
    with open('black_opcodes.txt', 'r') as f:
        for line in f:
            black_file_list.append(line.strip('\n'))

    with open('white_opcodes.txt', 'r') as f:
        for line in f:
            white_file_list.append(line.strip('\n'))
    old_data = white_file_list + black_file_list
    return old_data
```

在训练样本时，程序将所有样本分成了两部分，一部分拿去训练得到模型，一部分使用训练出来的模型拿来做预测，检测该模型的效果。当算法确定后，就需要将所有样本拿来训练获得最终的训练模型。

```py
def do_rf_fin(x, y):
    clf = RandomForestClassifier(n_estimators=50)
    clf.fit(x, y)
    joblib.dump(clf, 'model/rf_fin.pkl')
```

最后，我们再使用训练出的`rf_fin.pk1`模型做预测:

```py
if __name__ == '__main__':
    php_file_name = sys.argv[1]
    print 'Checking the file {}'.format(php_file_name)
    all_file = get_old_data()
    opcode = load_php_opcode(php_file_name)
    all_file.append(opcode)
    x = get_feature_by_tfidf(all_file)
    gnb = joblib.load('save/rf_fin.pkl')
    y_p = gnb.predict(x[-1:])
    if y_p == [0]:
        print 'Not Webshell'
    elif y_p == [1]:
        print 'Webshell!'
```

### 

### **0x06 优化检测率**

#### 1.优化数据集

本文主要从数据集的角度进行检测率的优化。优化数据集分为两方面，一方面是增大数据量，数据量越大，预测的效果越好，本次测试黑样本较少，另一方面是清洗数据集，主要针对的是黑样本的清洗。

在预测的过程中，发现模型会将空文件或者没有php代码的文件识别为webshell，这些文件的opcode序列为`ECHO RETURN`，回过头找黑名单中opcode序列为`ECHO RETURN`的webshell文件，发现很多webshell的代码使用的是php短标签`<? ?>`

，而php.ini没有配置短标签支持，导致这些短标签的webshell不能被php正常解析，最终和普通txt文件的opcode相同，产生了数据集污染，将正常的空文件或文本文件识别成了webshell。

如下图所示：

![](/assets/shell-10.png)

修改php.ini，将短标签解析开启short\_open\_tag = On，重新进行特征提取，发现还是存在少量ECHO RETURN序列，直接手工去除，再次训练。

使用朴素贝叶斯的效果如下，并没有提高准确率和召回率，还有所下降，说明朴素贝叶斯在此不适用。由于之前去掉了一部分黑名单样本导致黑白样本数量差距变大，导致朴素贝叶斯分类效果变差。

#### 2.优化算法

实践发现，基于这些数据集的情况下，随机森林算法的效果要优于朴素贝叶斯，优化算法可以使用如深度学习的CNN、RNN等算法进行尝试。

#### 3.优化参数

参数优化主要分为算法模型调参和特征提取调参，这里主要可调的参数为N-Gram数和词袋最大特征数。

这里主要介绍一下N-Gram，N-Gram模型是基于“联想”，它的一个特点是某个词的出现依赖于其他若干个词，第二个特点是我们获得的信息越多，预测越准确。例如听到腾讯就想到qq，听到百度，阿里就会想到腾讯，腾讯、qq就相当于2-Gram,百度、阿里、腾讯相当于3-Gram。

上文所使用的是3-Gram，其中参数`ngram_range`是指定ngram的范围，也可以使用`ngram_range=(2,3)`表示同时使用2-Gram和3-Gram。

```py
cv = CountVectorizer(ngram_range=(3, 3), decode_error="ignore",
                         max_features=max_features,token_pattern=r'\b\w+\b', 
                         min_df=1, max_df=1.0)
```

### 0x07  结语

最后咱们总结一下机器学习在Webshell 检测过程中的思路和操作。

1. 提取特征，准备数据
2. 找到合适的算法，进行训练
3. 检查是否符合心中预期，会不会出现过度拟合等常见的问题。
4. 提供更多更精准的数据，或更换算法。
5. 重复1~4



