ESAPI是owasp提供的一套API级别的web应用解决方案。简单的说，ESAPI就是为了编写出更加安全的代码而设计出来的一些API，方便使用者调用，从而方便的编写安全的代码

其官方网站为：[https://www.owasp.org/，其有很多针对不同语言的版本，其J2ee的版本需要jre1.5及以上支持](https://www.owasp.org/，其有很多针对不同语言的版本，其J2ee的版本需要jre1.5及以上支持)

相关api介绍：

[https://www.javadoc.io/doc/org.owasp.esapi/esapi/2.1.0](https://www.javadoc.io/doc/org.owasp.esapi/esapi/2.1.0)

####  {#安装篇}

#### 安装篇 {#安装篇}

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

在工程的资源文件目录下增加配置文件ESAPI.properties及validation.properties，文件内容可为空。如果为空则都取默认值。建议参考以下文件内容设置

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

##### 第三步：测试 {#第三步测试}

增加测试类：

```
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

#### 使用篇 {#使用篇}

##### 针对xss漏洞 {#针对xss漏洞}

```
//对用户输入“input”进行HTML编码，防止XSS
input = ESAPI.encoder().encodeForHTML(input);
//根据自己不同的需要可以选用以下方法
//input = ESAPI.encoder().encodeForHTMLAttribute(input);
//input = ESAPI.encoder().encodeForJavaScript(input);
//input = ESAPI.encoder().encodeForCSS(input);
//input = ESAPI.encoder().encodeForURL(input);
```

##### 针对sql注入漏洞 {#针对sql注入漏洞}

除了支持mysql还支持oracle

```
String input1="用户输入1";
String input2="用户输入2";

//解决注入问题
input1 = ESAPI.encoder().encodeForSQL(new MySQLCodec(MySQLCodec.Mode.STANDARD),input1);
input2 = ESAPI.encoder().encodeForSQL(new MySQLCodec(MySQLCodec.Mode.STANDARD),input2);

String sqlStr="select name from tableA where id="+input1 +"and date_created ='" + input2+"'";

//执行SQL
```

##### 验证输入 {#验证输入}

检查每个输入的有效性，让每个输入合法。这里面的关键是一切都进行验证。ESAPI提供了很多常见的校验，可以方便针对不同的需要做校验。

```
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

##### 验证恶意文件 {#验证恶意文件}

```
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



