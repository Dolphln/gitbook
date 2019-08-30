## **一、选择Burp Suite插件开发语言**

Burp Suite支持Java,Python,Ruby编写他的插件，在这里我们选用Python作为我们插件的开发语言，Python分很多种，常见的比如Jython，Cython等等。今天我们用的是Jython，Jython为我们提供了Python的库，同时也提供了所有java的类。

## 二、配置Jython环境

我们需要让Burp Suite加载我们的插件，在[https://www.jython.org/download](https://www.jython.org/download)（可下载Standalone独立jar包）。下载好后如下图使Burp Suite加载Python插件。

![](/assets/burp-1.png)

## 三、介绍API

Burp Suite官方API文档：[https://portswigger.net/burp/extender/api/index.html](https://portswigger.net/burp/extender/api/index.html)

从Burp Suite上我们可以直接查看api文档，也可下载到本地：

![](/assets/burp-12.png)

### 1.（红色）插件入口和帮助接口类：

```
IBurpExtender # Burp插件的入口

IBurpExtenderCallbacks # IBurpExtender接口的实现类

IExtensionHelpers     # 帮助接口

IExtensionStateListener # 管理操作接口
```

### 2.（绿色）UI相关接口类：

```
IContextMenuFactory

IContextMenuInvocation

ITab

ITextEditor

IMessageEditor

IMenuItemHandler

# 以上主要是定义Burp插件的UI显示和动作的处理事件，主要是软件交互中使用
```

### 3.（蓝色） Burp工具组件接口类：

```
IInterceptedProxyMessage

IIntruderAttack

IIntruderPayloadGenerator

IIntruderPayloadGeneratorFactory

IIntruderPayloadProcessor

IProxyListener

IScanIssue

IScannerCheck

IScannerInsertionPoint

IScannerInsertionPointProvider

IScannerListener

IScanQueueItem

IScopeChangeListener

#  Burp Suite工具组件接口类
```

### 4.（棕色）HTTP消息处理接口类：

```
ICookie

IHttpListener

IHttpRequestResponse

IHttpRequestResponsePersisted

IHttpRequestResponseWithMarkers

IHttpService

IRequestInfo

IParameter

IResponseInfo

# 处理Cookie、Request、Response、Parameter等消息头接口类
```

## 

## 四、**官方插件开发示例**

**官方给出了简单的插件示例，包括java，python，ruby版本**

[**https://portswigger.net/burp/extender\#SampleExtensions**](https://portswigger.net/burp/extender#SampleExtensions)

![](/assets/burp-13.png)

## 五、**实战开发**

目前jython还不支持python3，所以我们开发时还是采用python2的语法。

```py
from burp import IBurpExtender    
from burp import IIntruderPayloadGeneratorFactory    
from burp import IIntruderPayloadGenerator    
from java.util import List, ArrayList    
import random    


class BurpExtender(IBurpExtender, IIntruderPayloadGeneratorFactory):    
  def registerExtenderCallbacks(self, callbacks):    
    self._callbacks = callbacks    
    self._helpers = callbacks.getHelpers()    
    callbacks.setExtensionName("burp—plugin")    
    callbacks.registerIntruderPayloadGeneratorFactory(self)    
    return    

  def getGeneratorName(self):    
    return "burp—plugin"    

  def createNewInstance(self, attack):    
    return BHPFuzzer(self, attack)    

class BHPFuzzer(IIntruderPayloadGenerator):    
  def __init__(self, extender, attack):    
    self._extender = extender    
    self._helpers  = extender._helpers    
    self._attack   = attack    
    print "burp—plugin"    
    self.max_payloads = 1000    
    self.num_payloads = 0    

    return    


  def hasMorePayloads(self):    
    print "hasMorePayloads called."    
    if self.num_payloads == self.max_payloads:    
      print "No more payloads."    
      return False    
    else:    
      print "More payloads. Continuing."    
      return True    


  def getNextPayload(self,current_payload):    
    payload = "".join(chr(x) for x in current_payload)    
    payload = self.mutate_payload(payload)    
    self.num_payloads += 1    
    return payload    

  def reset(self):    
    self.num_payloads = 0    
    return    

  def mutate_payload(self,original_payload):    
    picker = random.randint(1,3)    
    offset  = random.randint(0,len(original_payload)-1)    
    payload = original_payload[:offset]    

    if picker == 1:    
      payload += "'"    

    if picker == 2:    
      payload += "<script>alert('xss');</script>";    

    if picker == 3:    
      chunk_length = random.randint(len(payload[offset:]),len(payload)-1)    
      repeater     = random.randint(1,10)    

      for i in range(repeater):    
        payload += original_payload[offset:offset+chunk_length]    

    payload += original_payload[offset:]    
    return payload
```



