ESAPI是owasp提供的一套API级别的web应用解决方案。简单的说，ESAPI就是为了编写出更加安全的代码而设计出来的一些API，方便使用者调用，从而方便的编写安全的代码

owasp top 10所对应的API接口

| owasp top 10 | owasp esapi |
| :--- | :--- |
| A1. Cross Site Scripting \(xss\) | Validator,    Encoder |
| A2. Injection Flaws | Encoder |
| A3. Malicious  File Execution | HTTPUtilities \(safe upload\) |
| A4. Insecure Direct Object Reference | AccessReferenceMap,  AccessController |
| A5. Cross Site Request Forgercy \(CSRF\) | User \(CSRF Token\) |
| A6. Leakage and Improper Error Handling | EnterpriseSecurityException , HTTPUtils |
| A7. Broken Authentication and Sessions | Authenticator, User , HTTPUtils |
| A8. Insecure Cryptographic Storage | Encryptor |
| A9. Insecure Communications | HTTPUtilities \(Secure Cookie ,Channel\) |
| A10. Failure to Restrict URL Access | AccessController |

其官方网站为：[https://www.owasp.org/，其有很多针对不同语言的版本，其J2ee的版本需要jre1.5及以上支持](https://www.owasp.org/，其有很多针对不同语言的版本，其J2ee的版本需要jre1.5及以上支持)

相关api介绍：

[https://www.javadoc.io/doc/org.owasp.esapi/esapi/2.1.0](https://www.javadoc.io/doc/org.owasp.esapi/esapi/2.1.0)

### 安装篇

#### 第一步：引入Jar

###### Maven {#maven}

```
<dependency>
    <groupId>org.owasp.esapi</groupId>
    <artifactId>esapi</artifactId>
    <version>2.1.0.1</version>
</dependency>
```

#### 第二步：加入配置文件

在工程的资源文件目录下（src/main/resources）增加配置文件ESAPI.properties及validation.properties，文件内容可为空。如果为空则都取默认值。建议参考以下文件内容设置

##### ESAPI.properties

    # 是否要打印配置属性,默认为true
    ESAPI.printProperties=true

    ESAPI.AccessControl=org.owasp.esapi.reference.DefaultAccessController
    ESAPI.Authenticator=org.owasp.esapi.reference.FileBasedAuthenticator
    ESAPI.Encoder=org.owasp.esapi.reference.DefaultEncoder
    ESAPI.Encryptor=org.owasp.esapi.reference.crypto.JavaEncryptor
    ESAPI.Executor=org.owasp.esapi.reference.DefaultExecutor
    ESAPI.HTTPUtilities=org.owasp.esapi.reference.DefaultHTTPUtilities
    ESAPI.IntrusionDetector=org.owasp.esapi.reference.DefaultIntrusionDetector
    ESAPI.Logger=org.owasp.esapi.reference.JavaLogFactory
    ESAPI.Randomizer=org.owasp.esapi.reference.DefaultRandomizer
    ESAPI.Validator=org.owasp.esapi.reference.DefaultValidator


    #===========================================================================
    # ESAPI Encoder
    Encoder.AllowMultipleEncoding=false
    Encoder.AllowMixedEncoding=false
    Encoder.DefaultCodecList=HTMLEntityCodec,PercentCodec,JavaScriptCodec


    #===========================================================================
    # ESAPI 加密模块
    Encryptor.PreferredJCEProvider=
    Encryptor.EncryptionAlgorithm=AES
    Encryptor.CipherTransformation=AES/CBC/PKCS5Padding
    Encryptor.cipher_modes.combined_modes=GCM,CCM,IAPM,EAX,OCB,CWC
    Encryptor.cipher_modes.additional_allowed=CBC
    Encryptor.EncryptionKeyLength=128
    Encryptor.ChooseIVMethod=random
    Encryptor.fixedIV=0x000102030405060708090a0b0c0d0e0f
    Encryptor.CipherText.useMAC=true
    Encryptor.PlainText.overwrite=true
    Encryptor.HashAlgorithm=SHA-512
    Encryptor.HashIterations=1024
    Encryptor.DigitalSignatureAlgorithm=SHA1withDSA
    Encryptor.DigitalSignatureKeyLength=1024
    Encryptor.RandomAlgorithm=SHA1PRNG
    Encryptor.CharacterEncoding=UTF-8
    Encryptor.KDF.PRF=HmacSHA256

    #===========================================================================
    # ESAPI Http工具

    HttpUtilities.UploadDir=C:\\ESAPI\\testUpload
    HttpUtilities.UploadTempDir=C:\\temp
    # Force flags on cookies, if you use HttpUtilities to set cookies
    HttpUtilities.ForceHttpOnlySession=false
    HttpUtilities.ForceSecureSession=false
    HttpUtilities.ForceHttpOnlyCookies=true
    HttpUtilities.ForceSecureCookies=true
    # Maximum size of HTTP headers
    HttpUtilities.MaxHeaderSize=4096
    # File upload configuration
    HttpUtilities.ApprovedUploadExtensions=.zip,.pdf,.doc,.docx,.ppt,.pptx,.tar,.gz,.tgz,.rar,.war,.jar,.ear,.xls,.rtf,.properties,.java,.class,.txt,.xml,.jsp,.jsf,.exe,.dll
    HttpUtilities.MaxUploadFileBytes=500000000
    # Using UTF-8 throughout your stack is highly recommended. That includes your database driver,
    # container, and any other technologies you may be using. Failure to do this may expose you
    # to Unicode transcoding injection attacks. Use of UTF-8 does not hinder internationalization.
    HttpUtilities.ResponseContentType=text/html; charset=UTF-8
    # This is the name of the cookie used to represent the HTTP session
    # Typically this will be the default "JSESSIONID" 
    HttpUtilities.HttpSessionIdName=JSESSIONID



    #===========================================================================
    # ESAPI Executor
    Executor.WorkingDirectory=
    Executor.ApprovedExecutables=


    #===========================================================================
    # ESAPI Logging
    # Set the application name if these logs are combined with other applications
    Logger.ApplicationName=ExampleApplication
    # If you use an HTML log viewer that does not properly HTML escape log data, you can set LogEncodingRequired to true
    Logger.LogEncodingRequired=false
    # Determines whether ESAPI should log the application name. This might be clutter in some single-server/single-app environments.
    Logger.LogApplicationName=true
    # Determines whether ESAPI should log the server IP and port. This might be clutter in some single-server environments.
    Logger.LogServerIP=true
    # LogFileName, the name of the logging file. Provide a full directory path (e.g., C:\\ESAPI\\ESAPI_logging_file) if you
    # want to place it in a specific directory.
    Logger.LogFileName=ESAPI_logging_file
    # MaxLogFileSize, the max size (in bytes) of a single log file before it cuts over to a new one (default is 10,000,000)
    Logger.MaxLogFileSize=10000000


    #===========================================================================
    # ESAPI Intrusion Detection
    IntrusionDetector.Disable=false
    IntrusionDetector.event.test.count=2
    IntrusionDetector.event.test.interval=10
    IntrusionDetector.event.test.actions=disable,log

    IntrusionDetector.org.owasp.esapi.errors.IntrusionException.count=1
    IntrusionDetector.org.owasp.esapi.errors.IntrusionException.interval=1
    IntrusionDetector.org.owasp.esapi.errors.IntrusionException.actions=log,disable,logout

    IntrusionDetector.org.owasp.esapi.errors.IntegrityException.count=10
    IntrusionDetector.org.owasp.esapi.errors.IntegrityException.interval=5
    IntrusionDetector.org.owasp.esapi.errors.IntegrityException.actions=log,disable,logout

    IntrusionDetector.org.owasp.esapi.errors.AuthenticationHostException.count=2
    IntrusionDetector.org.owasp.esapi.errors.AuthenticationHostException.interval=10
    IntrusionDetector.org.owasp.esapi.errors.AuthenticationHostException.actions=log,logout


    #===========================================================================
    # ESAPI 校验器
    #校验器的配置文件
    Validator.ConfigurationFile=validation.properties

    # Validators used by ESAPI
    Validator.AccountName=^[a-zA-Z0-9]{3,20}$
    Validator.SystemCommand=^[a-zA-Z\\-\\/]{1,64}$
    Validator.RoleName=^[a-z]{1,20}$

    #the word TEST below should be changed to your application 
    #name - only relative URL's are supported
    Validator.Redirect=^\\/test.*$

    # Global HTTP Validation Rules
    # Values with Base64 encoded data (e.g. encrypted state) will need at least [a-zA-Z0-9\/+=]
    Validator.HTTPScheme=^(http|https)$
    Validator.HTTPServerName=^[a-zA-Z0-9_.\\-]*$
    Validator.HTTPParameterName=^[a-zA-Z0-9_]{1,32}$
    Validator.HTTPParameterValue=^[a-zA-Z0-9.\\-\\/+=@_ ]*$
    Validator.HTTPCookieName=^[a-zA-Z0-9\\-_]{1,32}$
    Validator.HTTPCookieValue=^[a-zA-Z0-9\\-\\/+=_ ]*$

    # Note that max header name capped at 150 in SecurityRequestWrapper!
    Validator.HTTPHeaderName=^[a-zA-Z0-9\\-_]{1,50}$
    Validator.HTTPHeaderValue=^[a-zA-Z0-9()\\-=\\*\\.\\?;,+\\/:&_ ]*$
    Validator.HTTPContextPath=^\\/?[a-zA-Z0-9.\\-\\/_]*$
    Validator.HTTPServletPath=^[a-zA-Z0-9.\\-\\/_]*$
    Validator.HTTPPath=^[a-zA-Z0-9.\\-_]*$
    Validator.HTTPQueryString=^[a-zA-Z0-9()\\-=\\*\\.\\?;,+\\/:&_ %]*$
    Validator.HTTPURI=^[a-zA-Z0-9()\\-=\\*\\.\\?;,+\\/:&_ ]*$
    Validator.HTTPURL=^.*$
    Validator.HTTPJSESSIONID=^[A-Z0-9]{10,30}$

    # Validation of file related input
    Validator.FileName=^[a-zA-Z0-9!@#$%^&{}\\[\\]()_+\\-=,.~'` ]{1,255}$
    Validator.DirectoryName=^[a-zA-Z0-9:/\\\\!@#$%^&{}\\[\\]()_+\\-=,.~'` ]{1,255}$

    # Validation of dates. Controls whether or not 'lenient' dates are accepted.
    # See DataFormat.setLenient(boolean flag) for further details.
    Validator.AcceptLenientDates=false

##### validation.properties

```
# 校验某个字段的正则表达式
Validator.SafeString=^[.\\p{Alnum}\\p{Space}]{0,1024}$
Validator.Email=^[A-Za-z0-9._%'-]+@[A-Za-z0-9.-]+\\.[a-zA-Z]{2,4}$
Validator.IPAddress=^(?:(?:25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\\.){3}(?:25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)$
Validator.URL=^(ht|f)tp(s?)\\:\\/\\/[0-9a-zA-Z]([-.\\w]*[0-9a-zA-Z])*(:(0-9)*)*(\\/?)([a-zA-Z0-9\\-\\.\\?\\,\\:\\'\\/\\\\\\+=&amp;%\\$#_]*)?$
Validator.CreditCard=^(\\d{4}[- ]?){3}\\d{4}$
```

#### 

#### 第三步：测试

增加测试类：

```java
import org.owasp.esapi.ESAPI;
import org.owasp.esapi.Encoder;
import org.owasp.esapi.codecs.MySQLCodec;
import org.owasp.esapi.errors.EncodingException;


public class esapi_test {
    public static void main(String args[]) throws EncodingException {
        System.out.println(ESAPI.encoder().encodeForHTML("<a href='http://www.baidu.com'></a><script>alert(/xss/); </script>"));

        System.out.println(ESAPI.encoder().encodeForURL("http://www.baidu.com/?id=a&&age=11"));

        String sqlstr= "' or '1'='1";

        System.out.println(ESAPI.encoder().encodeForSQL(new MySQLCodec(MySQLCodec.Mode.STANDARD),sqlstr));   #mysql
    }
}
```

运行结果如下即为安装成功:

```
&lt;a href&#x3d;&#x27;sdfs&#x27;&gt;&lt;&#x2f;a&gt; &lt; script &gt; alert&#x28;&#x29;&#x3b; &lt;&#x2f; script &gt;
http%3A%2F%2Fwww.baidu.com%2F%3Fid%3Da%26%26age%3D11
\' or \'1\'\=\'1
```

####  {#使用篇}

### 使用篇

#### 1、针对xss漏洞

```java
//对用户输入“input”进行HTML编码，防止XSS
input = ESAPI.encoder().encodeForHTML(input);
//根据自己不同的需要可以选用以下方法
//input = ESAPI.encoder().encodeForHTMLAttribute(input);
//input = ESAPI.encoder().encodeForJavaScript(input);
//input = ESAPI.encoder().encodeForCSS(input);
//input = ESAPI.encoder().encodeForURL(input);
```

ESAPI提供了两个相关接口 Encode、Validator。

Encode接口

Encode（编码器接口）包含了许多解码输入和编码输出的方法，这样处理过的字符对于各种解释器都是安全的。

ESAPI根据XSS问题的特征和产生的原因，提供了不同的接口，下面我们介绍各个编码器的使用和原理。

Encode接口针对XSS的预发是在输出编码上，根据你要输出到不同地方转换成不同的编码。不同地方有HTML、Javascript、URL、CSS等。

（需了解3种编码格式：URL编码、HTML编码、JavaScript编码）其思想是对特殊意义的字符必须进行编码，编码后不影响原来结构体。

![](/assets/esapi-1.png)

列举ESAPI针对XSS漏洞的7个预防方法：

##### **\(1\).调用HTML编码器**

Code:

```java
System.out.println(ESAPI.encoder().encodeForHTML("<script>alert(/xss/)</script>"));
System.out.println(ESAPI.encoder().encodeForHTML("data 12"));
```

输出

![](/assets/esapi-2.png)

**总结：**

这个编码器使用的是HTMLEntityCodec，编译原理是，如果是空格、字母或者数字就不进行编码；如果有特殊字符在HTML中有匹配的替代字符，就使用替代字符。

**应用场景：**

有需要输出用户输入的字符串、表单提交后的数据需要展示的场景都可以运用。下面用留言板作为案例。

**Code:**

从数据库中得到数据，解析展示到用户浏览器界面。

这样即使用户输入了html标签，在返回到浏览器的时候也得不到执行。因为已经转换成html编码了。

##### **（2）调用HTML属性编码器**

code:

```java
System.out.println(ESAPI.encoder().encodeForHTMLAttribute("<script>alert(/xss/)</script>"));

System.out.println(ESAPI.encoder().encodeForHTMLAttribute("data 12"));
```

![](/assets/esapi-3.png)

**总结：**

```
     HTML属性编码和HTML编码在实现原理上是一样的，唯一的不同点就是HTML属性编码需要对空格进行编码。所以也就是免疫了一个空格而已。
```

##### （3）调用JavaScript编码器

code:

```java
System.out.println(ESAPI.encoder().encodeForJavaScript("<script>alert(/xss/)</script>"));

System.out.println(ESAPI.encoder().encodeForJavaScript("data 12"));
```

![](/assets/esapi-4.png)

总结：

```
   JavaScript编码器首先判断是不是免疫的字符，如果是免疫字符，就直接返回；如果是数字、字母也直接返回；如果是小于256的字符就使用\xHH的编码方式；如果是大于256的字符，就使用\uHHHHH的方式编码。
```

##### （4）调用CSS编码器

code:

```java
System.out.println(ESAPI.encoder().encodeForCSS("<script>alert(/xss/)</script>"));

System.out.println(ESAPI.encoder().encodeForCSS("data 12"));
```

![](/assets/esapi-5.png)

总结：

```
      ESAPI的CSS编码是通过反斜杠（\）加上十六进制数进行的编码。会对空格、特殊字符都进行编码。
```

##### （5）调用URL编码器

code:

```java
System.out.println(ESAPI.encoder().encodeForURL("http://www.baidu.com/?id=1&page=1"));

System.out.println(ESAPI.encoder().encodeForURL("http://www.baidu.com/?callback=<script>alert('xss')</script>"));
```

![](/assets/esapi-6.png)

总结：

```
    URL编码器先将字符串转换为UTF-8，然后对转换的字符串用%加上十六进制数的方式编码。
```

##### （6）调用复合\(嵌套\)编码器

code:

```java
System.out.println(ESAPI.encoder().encodeForJavaScript(ESAPI.encoder().encodeForHTML("<a href='sdfs'></a> < script > alert(); </ script >")));
```

总结：

```
  上面那几种都只能处理一种编码的情况，对于存在多种编码的情况，可以使用复合\(嵌套\)编码，它根据由内到外的调用顺序来执行ESAPI的编码方法。
```

##### （7）小结

```
     由于Web页面上输出内容的地方很多，输出的背景环境也不相同，甚至有同一个输入可能输出到同一个页面上的不同地方，每一个地方的编码也可能不同，所以想彻底预防XSS是很困难的。从安全的角度考虑，建议使用HTTPOnly标志。
```

##### （8）Validator接口

```
    Validator接口定义了一组用于规范化和验证不可信输入的方法。开发人员可以扩展这个接口以适应自己的数据格式。与抛出异常相比，这个接口返回布尔结果，因为并非所有的验证问题都是安全问题。布尔返回允许开发人员更清晰地处理有效、无效的结果。
```

开发人员使用这个方法的时候必须采用“白名单”方法来验证特定的模式或字符集匹配。因为“黑名单”有可能被绕过。

```
    调用getValidInput\( \)
```

```java
// 1
getValidInput(java.lang.String context, java.lang.String input, java.lang.String type, int maxLength,boolean allowNull, ValidationErrorList errors);

// 2
isValidInput(java.lang.String context, java.lang.String input, java.lang.String type, int maxLength,boolean allowNull);

String validatedFirstName =ESAPI.validator().getValidInput("FirstName", myForm.getFirstName(), "FirstNameRegex", 255, false, errorList);

// 3
boolean isValidFirstName = ESAPI.validator().isValidInput("FirstName", myForm.getFirstName(), "FirstNameRegex", 255, false);
```

应用场景：

在有数据提交的地方，防止跨站攻击。 解决方案就是验证输入，检查每个输入的有效性。

```java
String input = "<script>alert('xss')</script>";
String type = "SafeString"; //使用validation.properties文件定义的SafeString正则表达式规则
try {
    String data = ESAPI.validator().getValidInput(
            "Swingset Validation Secure Exercise", input, type,
            200, false);
    System.out.println(data);
} catch (ValidationException e) {
    /*当检测到输入字符串不匹配正则表达式时产生该异常，应进行适当处理*/
    input = "Validation attack detected";
    request.setAttribute("userMessage", e.getUserMessage());
    request.setAttribute("logMessage", e.getLogMessage());
} catch (Exception e) {
    input = "exception thrown";
    request.setAttribute("logMessage", e.getMessage());
}
```

返回规范化和验证过的输入数据。所有无效的输入将生成一个描述性的ValidationException异常。

#### 2、针对注入类漏洞

```
       防治Injection Flaws类型的漏洞有多种类型比如有命令注入、XPath注入、LDAP注入、SQL注入、其它注入，我举例一个SQL注入的防护，而针对SQL注入ESAPI提供了一个相关接口Encode。
```

Encode接口 1.调用 ESAPI.Encoder\(\).encodeForSQL\(\)

除了支持mysql还支持oracle

```java
String input1="用户输入1";
String input2="用户输入2";

//解决注入问题
input1 = ESAPI.encoder().encodeForSQL(new MySQLCodec(MySQLCodec.Mode.STANDARD),input1);
input2 = ESAPI.encoder().encodeForSQL(new MySQLCodec(MySQLCodec.Mode.STANDARD),input2);

String sqlStr="select name from tableA where id="+input1 +"and date_created ='" + input2+"'";

//执行SQL
```

**总结：**

可以看出使用ESAPI防范SQL注入非常容易，只需要创建一个相应的数据库编码器，然后在调用ESAPI.encoder\(\).encodeForSQL时，作为第一个参数传入即可。 ESAPI也是封装好了过滤规则。

应用场景：

比如在搜索、查询场景下，需要用户输入的字符串插入SQL命令的地方，就可以运用。

#####  {#验证输入}

##### 验证输入 {#验证输入}

检查每个输入的有效性，让每个输入合法。这里面的关键是一切都进行验证。ESAPI提供了很多常见的校验，可以方便针对不同的需要做校验。

```java
//type就是定义在validate.properties文件中的正则表达式
//ESAPI.validator().getValidInput(java.lang.String context,java.lang.String input, java.lang.String type,int maxLength,boolean allowNull);

//ESAPI.validator().isValidInput(java.lang.String context,java.lang.String input, java.lang.String type,int maxLength,boolean allowNull);

//实际使用参考如下：
String input="xxxx.com";

        if(!ESAPI.validator().isValidInput("",input,"Email",11,false)){
            System.out.println("出错了");
        }

        try{

            input = ESAPI.validator().getValidInput("",input,"Email",11,false);
        }catch (Exception e){
            System.out.println("输入错误");
            e.printStackTrace();
        }
```

#### 3、恶意文件执行类漏洞

```
    防治Malicious File Execution类型的漏洞，ESAPI提供了一个相关接口 Validator。

    Validator接口

     使用ESAPI防治恶意文件执行漏洞有2个方法，1是使用ESAPI验证上传文件名。2是检查文件大小。
```

#####  {#验证恶意文件}

##### （1）调用isValidFileName\(\) {#验证恶意文件}

```java
//校验文件名
String inputfilename ="xxxx.txt";
ArrayList<String> allowedExtension = new ArrayList<String>();
allowedExtension.add("exe");
allowedExtension.add("jpg");

if(!ESAPI.validator().isValidFileName("upload",inputfilename, allowedExtension,false)){
   //文件名不合法,throw exception
   System.out.println("出错了");
}

//获取随机文件名
System.out.println(ESAPI.randomizer().getRandomFilename("exe"));
//得到结果rnQO8AK4ymmv.exe
```

##### （2）使用ESAPI检查上传文件大小

```java
ServletFileUpload upload=newServletFileUpload(factory);
upload.setSizeMax(maxBytes)；
```

#### 4、不安全的直接对象引用类漏洞

防治Insecure Direct Object Reference类型的漏洞，ESAPI提供了两个相关类AccessReferenceMap、AccessController。

AccessReferenceMap类

AccessReferenceMap是ESAPI提供的用户实现非直接引用ID和直接引用ID之间匹配的类。

##### （1）.实例化RandomAccessReferenceMap类

```java
//使用随机访问的引用map
AccessReferenceMapmap = new RandomAccessReferenceMap();

String userID =arm.getDirectReference(indirectRef);
```

应用场景：

在需要传递传递的访问中，比如用户访问信息页，通过UserID的操作。

AccessController接口

角色权限验证，检查当前用户对应的角色是否有权限访问url，权限的定义在esapi\fbac-policies\URLAccessRules.txt文件中。

1.调用assertAuthorizedForURL\(\)

```java
ESAPI.accessController().assertAuthorizedForURL();
```

#### 5、CSRF类漏洞

```
    防治Cross Site Request Forgery类型的漏洞，最常用的解决方案就是使用Token，在任何需要保护的表单，增加一个隐藏的字段来存放这个Token。对于需要保护的URL，需要增加一个参数来存放这个Token。ESAPI提供了一个相关接口 User。
```

User接口

管理身份验证和身份管理的规则接口。

1.调用resetCSRFToken\(\)

使用流程:

a.新建CSRF令牌添加进用户每次登陆以及存储在http session里，这种令牌至少对每用户会话来说应该是唯一的，或者是对每个请求是唯一的。

```java
//产生一个新的CSRF Token，在用户第一次登录的时候保存在用户的session中。
private String csrfToken = resetCSRFToken();
public String resetCSRFToken() {
    csrfToken = ESAPI.randomizer().getRandomString(8, DefaultEncoder.CHAR_ALPHANUMERICS);
    return csrfToken;
}
```

b.令牌同样可以包含在URL中或作为一个URL参数标记/隐藏字段。

c.在服务器端检查提交令牌与用户会话对象令牌是否匹 配。

d.在注销和会话超时，删除用户对象会话和会话销毁。

总结：

用ESAPI防止CSRF攻击的关键点就是生成Token，而且不能被攻击者知道生成方式，如果攻击者能伪造你的Token说明也是不安全的。还有CSRF安全的前提是你的站点没有XSS，不然攻击者利用XSS漏洞一样可以绕过CSRF检测。

应用场景：

在登录用户进行的操作里，都可以加上Token，比如用户提交修改密码操作。

#### 6、安全配置错误类漏洞

防治Leakage and Improper Error Handling类型的漏洞，需要具体项目具体分析。

Enterprise Security Exception

据不完全统计，国内个人信息泄露数达55.3亿条左右，平均每人就有4条相关的个人信息泄露，这些信息最终的命运，是在黑市中反复倒手，直至被榨干价值。其中，80％的数据泄露自企业内鬼，黑客仅占20％。

上面的信息说明（统计没有数据来源支持，不可全信），不过这类问题基本是内部自身的问题，得从内部开始查找、解决。

#### 7、 失效的身份认证和会话管理类漏洞

无

#### 8、不安全的加密存储类漏洞

防治Insecure Cryptographic Storage类型的漏洞，ESAPI提供了一个相关接口 Encryptor。

Encryptor接口

Encryptor接口提供了一组用于执行通用加密、随机数和散列操作的方法。在配置文件ESAPI.properties中提供了关于加密的配置项，可以设置使用的加密算法、秘钥长度、初始化向量、哈希和签名算法。

1.调用encrypt（）

```java
//加密
String ciphertext =ESAPI.encryptor().encrypt("123456");
```

2.调用decrypt \(\)

```java
//解密
String decrypted =ESAPI.encryptor().decrypt(ciphertext);
```

个人总结：

解密所提供的密文，使用来自它的信息和由属性Encryptor指定的主加密密钥。

#### 9、 传输层保护不足类漏洞

防治Insecure Communications类型的漏洞，预防措施有三种：

一、秘钥交换，

二、对称加密与非对称加密结合，

三、SSL、TLS。

#### 

#### 10、 未限制URL访问类漏洞

防治Failure to Resstrict URL Access类型的漏洞，ESAPI提供了一个相关接口 AccessController。

AccessController接口

AccessController接口定义了一组方法，这些方法可以在各种应用程序中使用，以加强访问控制。如果这个URL不是公开的，那么必须限制能够访问他的授权用户，加强基于用户或角色的访问控制，完全禁止访问未被授权的页面类型（如配置文件、日志文件、源文件等）

1.调用assertAuthorizedForURL\(Stringurl\)

Codemode：

ESAPI.accessController\(\).assertAuthorizedForURL\(request.getRequestURI\(\).toString\(\)\);

个人总结：

该方法检查当前用户是否被授权访问引用的URL。通常该方法在应用程序的控制器或过滤器中调用。

参考：

[http://liehu.tass.com.cn/archives/1427](http://liehu.tass.com.cn/archives/1427)

[https://blog.csdn.net/frog4/article/details/81876462](https://blog.csdn.net/frog4/article/details/81876462)

[https://wenku.baidu.com/view/c24d4e642af90242a895e588.html](https://wenku.baidu.com/view/c24d4e642af90242a895e588.html)

