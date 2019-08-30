**今天将利用python实战写一个简单的sql注入插件（实现每个参数后面加入单引号），开发过程我会详细介绍每一个步骤，后续可模仿着，完善插件。**

**所需要的接口类：**

![](/assets/burp-21.png)

**IBurpExtender**：

       所有插件必须实现这个接口，类名字必须为“BurpExtender”，并且必须提供一个默认构造器”。

IBurpExtender用来在burp上面注册扩展，IBurpExtender里面还有一个registerExtenderCallbakcs类方法需要实现：         

![](/assets/burp-22.png)

![](/assets/burp-23.png)

当扩展被调用时，会注册一个IBurpExtenderCallbacks实例，该实例提供了许多常用操作：

        此接口中实现的方法和字段在插件开发过程中会经常使用到。 Burp Suite 利用此接口向扩展中传递了许多回调方法，这些回调方法可被用于在 Burp 中执行多个操作。当扩展被加载后，Burp 会调用**registerExtenderCallbacks\(\)**方法，并传递一个**IBurpExtenderCallbacks**的实例。**扩展插件可以通过这个实例调用很多扩展 Burp 功能必需的方法**。如：设置扩展插件的属性，操作 HTTP 请求和响应以及启动其他扫描功能等等。

![](/assets/burp-24.png)

先完成和理解部分代码：

![](/assets/burp-25.png)



**IIntruderPayloadGeneratorFactory**:

调用IBurpExtenderCallbacks.registerintruder

PayloadGeneratorFactory\(\)注册一个payload生成器。

此类下面有两个类方法需要实现“createNewInstance”和“getGeneratorName”

![](/assets/burp-26.png)

createNewInstance方法：创建一个payload生成器新的实例，发动插件攻击时会返回payload生成器的实例。

getGeneratorName方法：用来获取payload生成器的名称

继续完成和理解代码：

![](/assets/burp-27.png)



我们已经注册了payload生成器，现在我们需要用一个接口类去定义我们的payload生成器

**IIntruderPayloadGenerator：**这个接口类用来定义插件的payload生成器，定义的前提是我们得有东西去定义。所以我们用IIntruderPayloadGeneratorFactory返回此接口的新实例。

这个接口类里面有三个类方法”getNextPayload”,”hasMorePayloads”,”reset”

![](/assets/burp-28.png)

getNextPayload：用于获取下一个payload

hasMorePayloads：决定生成器是否能够提供更多payload

reset ：重制生成器状态，使下次调用getNextPayload方法时返回第一条payload

继续完成和理解代码：

![](/assets/burp-29.png)

我们可以打印出current\_payload和转码后的payload看看：

![](/assets/burp-210.png)

![](/assets/burp-211.png)



这里就不做过多解释了，一目了然。我这里使用的DVWA-low-sql的环境进行的测试。

贴一张完整的简洁的代码：

![](/assets/burp-212.png)

最后再附一张图整理逻辑：

![](https://image.3001.net/images/20190110/1547096393_5c36d1497423d.png!small " 利用python开发Burp Suite插件")



