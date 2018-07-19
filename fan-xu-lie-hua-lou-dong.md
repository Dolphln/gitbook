## 0x01 概念

序列化：把对象转换为字节序列的过程。

反序列化：把字节序列恢复为对象的过程。![](/assets/反序列化概念.png)

### 0x01.1 关键api

        java.io.ObjectOutputStream代表对象输出流，它的writeObject\(Object obj\)方法可对参数指定的obj对象进行序列化，把得到的字节序列写到一个目标输出流中。

        java.io.ObjectInputStream代表对象输入流，它的readObject\(\)方法从一个源输入流中读取字节序列，再把它们反序列化为一个对象，并将其返回。

### [Java序列化和反序列化](https://xz.aliyun.com/t/1825)

### [java反序列化漏洞从入门到深入](https://xz.aliyun.com/t/2041#toc-6)

### [先知议题解读 \| Java反序列化实战](https://www.anquanke.com/post/id/148593)



