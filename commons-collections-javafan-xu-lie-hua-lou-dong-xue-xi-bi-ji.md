# 前言

在2015年11月6日FoxGlove Security安全团队的@breenmachine 发布了一篇长博客里，借用Java反序列化和Apache Commons Collections这一基础类库实现远程命令执行的真实案例来到人们的视野，各大Java Web Server纷纷躺枪，这个漏洞横扫WebLogic、WebSphere、JBoss、Jenkins、OpenNMS的最新版。而在将近10个月前， Gabriel Lawrence 和Chris Frohoff 就已经在AppSecCali上的一个报告里提到了这个漏洞利用思路。　



目前，针对这个"2015年最被低估"的漏洞，各大受影响的Java应用厂商陆续发布了修复后的版本，Apache Commons Collections项目也对存在漏洞的类库进行了一定的安全处理。但是网络上仍有大量网站受此漏洞影响。

# java反序列化概念

　　序列化就是把对象的状态信息转换为字节序列\(即可以存储或传输的形式\)过程  
　　反序列化即逆过程，由字节流还原成对象  
　　注： 字节序是指多字节数据在计算机内存中存储或者网络传输时各字节的存储顺序。

**用途：**

　　1） 把对象的字节序列永久地保存到硬盘上，通常存放在一个文件中；  
　　2） 在网络上传送对象的字节序列。

**应用场景：**

　　1\) 一般来说，服务器启动后，就不会再关闭了，但是如果逼不得已需要重启，而用户会话还在进行相应的操作，这时就需要使用序列化将session信息保存起来放在硬盘，服务器重启后，又重新加载。这样就保证了用户信息不会丢失，实现永久化保存。

　　2\) 在很多应用中，需要对某些对象进行序列化，让它们离开内存空间，入住物理硬盘，以便减轻内存压力或便于长期保存。

　　　 比如最常见的是Web服务器中的Session对象，当有 10万用户并发访问，就有可能出现10万个Session对象，内存可能吃不消，于是Web容器就会把一些seesion先序列化到硬盘中，等要用了，再把保存在硬盘中的对象还原到内存中。

　　例子： 淘宝每年都会有定时抢购的活动，很多用户会提前登录等待，长时间不进行操作，一致保存在内存中，而到达指定时刻，几十万用户并发访问，就可能会有几十万个session，内存可能吃不消。这时就需要进行对象的活化、钝化，让其在闲置的时候离开内存，将信息保存至硬盘，等要用的时候，就重新加载进内存。

        Java中的`ObjectOutputStream`类的`writeObject()`方法可以实现序列化，类`ObjectInputStream`类的`readObject()`

方法用于反序列化。下面是将字符串对象先进行序列化，存储到本地文件，然后再通过反序列化进行恢复的样例代码：![](/assets/apache-java1.png)**序列号之后的数据解析**

数据开头为“AC ED 00 05”，数据内容包含了包名与类名、类中包含的变量名称、类型及变量的值。![](/assets/apache-java2.png)这里需要注意的是，`ac ed 00 05`是java序列化内容的特征，如果经过base64编码，那么相对应的是`rO0AB`：

![](/assets/apache-java3.png)





参考资料：

[https://www.vulbox.com/knowledge/detail/?id=11](https://www.vulbox.com/knowledge/detail/?id=11)

[https://xz.aliyun.com/t/1825](https://xz.aliyun.com/t/1825)

[https://xz.aliyun.com/t/136\#toc-1](https://xz.aliyun.com/t/136#toc-1)

[https://blog.chaitin.cn/2015-11-11\_java\_unserialize\_rce/?from=timeline&isappinstalled=0\#h2\_java%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96%E6%BC%8F%E6%B4%9E%E7%AE%80%E4%BB%8B](https://blog.chaitin.cn/2015-11-11_java_unserialize_rce/?from=timeline&isappinstalled=0#h2_java%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96%E6%BC%8F%E6%B4%9E%E7%AE%80%E4%BB%8B)

[https://www.iswin.org/2015/11/13/Apache-CommonsCollections-Deserialized-Vulnerability/](https://www.iswin.org/2015/11/13/Apache-CommonsCollections-Deserialized-Vulnerability/)



