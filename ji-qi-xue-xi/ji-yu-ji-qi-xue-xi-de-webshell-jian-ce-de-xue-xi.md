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





