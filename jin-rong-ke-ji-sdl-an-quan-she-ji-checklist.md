[https://mp.weixin.qq.com/s/MR3SmOLj834LK4RBMcZ2pg](https://mp.weixin.qq.com/s/MR3SmOLj834LK4RBMcZ2pg)



**1、验证码暴力破解**

手机或者邮箱验证码可枚举破解。

参考案例：[http://wooyun.jozxing.cc/static/bugs/wooyun-2013-029132.html](http://wooyun.jozxing.cc/static/bugs/wooyun-2013-029132.html)

**2、验证码直接返回**

服务器直接将验证码返回，例如某APP找回密码返回包。

![](/assets/1.png)

另外还有返回在Cookie中，例如

![](/assets/2.png)

另外还有可能在返回的页面源码中，需要注意提交的每一个参数。

参考链接：[http://wooyun.jozxing.cc/static/bugs/wooyun-2012-04728.html](http://wooyun.jozxing.cc/static/bugs/wooyun-2012-04728.html)

密码找回问题的答案隐藏在HTML中。

**3、跳过验证步骤**

输入重置账号后，跳过验证步骤，直接访问重置密码页面。

参考案例：[http://wooyun.jozxing.cc/static/bugs/wooyun-2013-042404.html](http://wooyun.jozxing.cc/static/bugs/wooyun-2013-042404.html)

**4、利用邮箱、手机号绑定**

绑定用户邮箱的时候，可以通过修改uid将其他用户的邮箱绑定为自己的，从而任意重置用户密码。

参考案例：[http://wooyun.jozxing.cc/static/bugs/wooyun-2012-08307.html](http://wooyun.jozxing.cc/static/bugs/wooyun-2012-08307.html)

**5、三方登录绑定其他用户**

例如绑定微博账号的时候，先登录微博并且获取code，然后绑定code和UID的请求如下：

thirdPartyType=1&uid=60570181&state=test&code=fb3a6454736e15534486c5a214067943

通过修改uid可以绑定将自己的微博绑定到其他用户账号，然后通过微博登录就可以登录任意用户账号。

**6、没有验证验证码接受手机号是否与username或者session一致**

例如发送验证码的请求如下：

username=\*\*\*&mobilePhone=\*\*\*&randcodeAjax=6119

没有验证username是否和mobilePhone一致，通过修改mobilePhone为自己的手机号接受验证码。

**7、本地验证绕过**

客户端在本地进行验证码是否正确的判断，而该判断结果也可以在本地修改，最终导致欺骗客户端，误以为我们已经输入了正确的验证码。

例如将返回包中的0修改为1即可绕过验证。

**8、重置密码最后一步uid或者username可控**

重置密码最后一步，重置账户通过用户传递的uid或者username控制，导致修改该UID即可重置其他用户密码。

**9、个人中心修改密码逻辑错误**

当前密码的校验和修改新密码是单独分开的两个包。所以可以理解为没有校验当前密码。

修改密码的请求中如下：

ssouid=\*\*\*&passwd=\*\*\*

修改该ssouid即可。

**10、利用session重新绑定用户**

重置密码最后一步是通过session获取用户名，然后再重置。而用户名是在重置密码第一步时与session进行绑定，那么如果重置密码的最后一步程序并没有验证该用户是够走完了验证流程，那么就可以通过重新绑定session为其他账号从而达到任意密码重置目的。

参考我之前提的案例：[http://wooyun.jozxing.cc/static/bugs/wooyun-2015-0114804.html](http://wooyun.jozxing.cc/static/bugs/wooyun-2015-0114804.html)

**11、接口暴露导致登录任意账户**

测试过程中发现\*\*\*.com/uc/md5Encode?encodeStr=这个连接会返回一个加密串pkey，而测试使用微博账号登录时，微博登录成功后，接口会返回的Oauth Code

然后站点会跳转到

\*\*\*.com/api/thirdpart/backinvoke.shtm?thirdPartyType=1&state=test&code=ab8be935aba7d6857\*\*\*\*64523524218

会返回一个自动提交的表单，内容如下：

![](/assets/3.png)

可以看到这个是防篡改了做了校验，如果我修改该username的值那么pkey的值就不匹配，无法成功登录。

而我们知道这个暴露了一个加密接口，我们组合上述字段的值提交给该接口发现pkey是吻合的。这就意味着防篡改验证失效，我可以通过修改username和该pkey值登录任意账户。

**12、重置密码Token可控**

重置密码最后一步的URL中的Key在之前的步骤中可以获得。

参考案例：[http://wooyun.jozxing.cc/static/bugs/wooyun-2014-058210.html](http://wooyun.jozxing.cc/static/bugs/wooyun-2014-058210.html)

[http://wooyun.jozxing.cc/static/bugs/wooyun-2015-094242.html](http://wooyun.jozxing.cc/static/bugs/wooyun-2015-094242.html)

**13、去掉验证参数绕过验证**

邮件系统取回密码功能设计逻辑错误，存在认证绕过漏洞，通过抓取数据包可通过修改报文，将找回问题答案参数删除后，直接进行对密码更改

参考案例：[http://wooyun.jozxing.cc/static/bugs/wooyun-2014-088927.html](http://wooyun.jozxing.cc/static/bugs/wooyun-2014-088927.html)

**14、邮箱找回Token可预测**

参考案例：[http://wooyun.jozxing.cc/static/bugs/wooyun-2015-090226.html](http://wooyun.jozxing.cc/static/bugs/wooyun-2015-090226.html)

token是当前的时间

参考案例：[http://wooyun.jozxing.cc/static/bugs/wooyun-2012-08333.html](http://wooyun.jozxing.cc/static/bugs/wooyun-2012-08333.html)

token为时间戳MD5值

参考案例：[http://wooyun.jozxing.cc/static/bugs/wooyun-2013-027138.html](http://wooyun.jozxing.cc/static/bugs/wooyun-2013-027138.html)

token为md5\($code.$email\)

**15、邮箱找回Token未与用户ID绑定**

参考案例：[http://wooyun.jozxing.cc/static/bugs/wooyun-2013-024956.html](http://wooyun.jozxing.cc/static/bugs/wooyun-2013-024956.html)

虽然有Token，但是没有和用户绑定，仍然可以通过修改username重置其他用户密码。

**16、绑定手机和邮箱处CSRF**

没有Token和验证码

参考案例：[http://wooyun.jozxing.cc/static/bugs/wooyun-2013-028893.html](http://wooyun.jozxing.cc/static/bugs/wooyun-2013-028893.html)

