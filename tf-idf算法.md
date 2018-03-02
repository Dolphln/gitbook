**TF-IDF是一种**[**统计方法**](http://link.zhihu.com/?target=http%3A//baike.baidu.com/item/%E7%BB%9F%E8%AE%A1%E6%96%B9%E6%B3%95)**，用以评估一字词对于一个文件集或一个语料库中的其中一份文件的重要程度**

阮一峰的《TF-IDF与余弦相似性的应用（一）：自动提取关键词》，解释的很浅显易懂，容易理解。

[http://www.ruanyifeng.com/blog/2013/03/tf-idf.html](http://www.ruanyifeng.com/blog/2013/03/tf-idf.html)

**词频TF**：文档中一个词语出现的次数，因为每篇文章长短不一样，所以需要对词频进行“标准化”![](/assets/词频.png)

**逆文档频率IDF**

```
    词频是统计一个词语出现的次数，但是一个词语的重要性不能仅仅使用词频来统计，比如“我”，“你”，“的”这些词语（成为停用词），在一个文档中的重要性起不到作用，表达不了很强的意识，所以需要对一个词语进行“重要性”的分配，即权重

    用统计学语言表达，就是在词频的基础上，要对每个词分配一个"重要性"权重。最常见的词（"的"、"是"、"在"）给予最小的权重，较常见的词（"中国"）给予较小的权重，较少见的词（"蜜蜂"、"养殖"）给予较大的权重。这个权重叫做"逆文档频率"（Inverse Document Frequency，缩写为IDF），它的大小与一个词的常见程度成反比![](/assets/逆文档频率.png)
```

![](/assets/逆文档频率.png)

**一个词语的TF-IDF值就是词频TF和逆文档频率IDF的相乘，某个词对文章的重要性越高，它的TF-IDF值就越大。**![](/assets/TF-IDF.png)

_**一个词语在一个稳定中的重要性随着它在文档中出现的次数成正比增加，但会随着它在语料库中出现的频率成反比下降。**_

获取语料库的一个案例

[http://blog.csdn.net/lionel\_fengj/article/details/53699903](http://blog.csdn.net/lionel_fengj/article/details/53699903)

https://zhuanlan.zhihu.com/p/27330205?utm\_source=tuicool&utm\_medium=referral

