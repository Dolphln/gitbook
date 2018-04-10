**在逻辑漏洞中，任意用户密码重置最为常见，可能出现在新用户注册页面，也可能是用户登录后重置密码的页面，或者用户忘记密码时的密码找回页面。其中，密码找回功能是重灾区。我把日常渗透过程中遇到的案例作了漏洞成因分析，这次，关注因重置凭证泄漏导致的任意用户密码重置问题。**

## **案例一**

用邮件找回密码时，作为重置凭证的验证码在 HTTP 应答中下发客户端，抓包后可轻易获取。先用攻击者账号走一次密码找回流程，测试账号 yangyangwithgnu@yeah.net 选用邮箱找回密码：

![](/assets/密码重置11.png)

点击获取校验码后抓取如下应答：

![](/assets/密码重置12.png)

其中，VFCode 从字面理解很可能是校验码。登录邮箱查看网站发过来的密码找回邮件：

![](/assets/密码重置13.png)

发现两者一致，那么，几乎可以确认服务端将密码找回的校验码泄漏至客户端，可导致任意账号密码重置问题。

尝试找回普通账号的密码。密码找回首页输入邮箱后，系统将立即校验该邮箱是否注册：

![](/assets/密码重置14.png)

将 UName 参数定义为枚举变量，以常见 qq 邮箱作为字典，可枚举出多个有效邮箱：

![](/assets/密码重置15.png)

以 chenwei@qq.com 为例，在应答包中找到校验码，成功将其密码重置为 PenTest1024，验证可登录：

![](/assets/密码重置16.png)

尝试找回管理员账号的密码。从该网站的域名注册信息中找到联系人的邮箱为 fishliu@xxxx.cn，可推测后台用户的邮箱后缀为 @xxxx.cn，所以，用常见后台用户名简单调整可构造出后台用户邮箱字典，枚举出大量后台用户：

![](/assets/密码重置17.png)

同理可重置这些后台用户的账号密码，为避免影响业务，不再实际操作。



**案例二**

用邮件找回密码时，带凭证的重置链接泄漏至客户端，抓捕可获取。用攻击者账号走一次密码找回流程。在找回密码页面输入攻击者账号及其邮箱（yangyangwithgnu、yangyangwithgnu@yeah.net）后提交：  
![](/assets/密码重置18.png)





拦截如下应答：

![](/assets/密码重置19.png)



显然是个重定向，isVerify、PassPhrase 这两个参数很可疑，后续交互中应留意，先放包，进入发送重置邮件的页面，输入验证码后提交。登录攻击者邮箱查看重置邮件：

![](/assets/密码重置120.png)

这个带 token 的重置链接似曾相识，对，就是前面抓包获取的 token 信息，比对看下：

```
forgotPwdEa.php?isVerify=eWFuZ3lhbmd3aXRoZ251fHlhbmd5YW5nd2l0aGdudUB5ZWFoLm5ldHw2MzQyNDkw
&
PassPhrase=01e4f6d4ede81b2604dc320bc4e3a6e8
```

```
forgotPwdEc.php?isVerify=eWFuZ3lhbmd3aXRoZ251fHlhbmd5YW5nd2l0aGdudUB5ZWFoLm5ldHw2MzQyNDkw
&
PassPhrase=01e4f6d4ede81b2604dc320bc4e3a6e8
```

唯一区别 forgotPwdEa 和 forgotPwdEc 两个文件名。

接下来验证通过服务端泄漏的 token 能否重置普通用户的账号密码。从重置流程可知，要重置密码必须提供用户名及其邮箱（或手机号）。

获取有效用户名。注册页面中，输入用户名后立即校验该用户名是否被占用：

![](/assets/密码重置121.png)

对应请求、应答如下：

![](/assets/密码重置122.png)



用户名已存在返回 failed，不存在返回 ok。以此特征，用常见国人姓名字典，可枚举出大量有效用户名（如 chenchuan、chenanqi、chenanxiu、zhangfeng 等等），存为 username.txt。

获取有效用户名对应邮箱。密码找回首页提交的请求中，user\_name 与 email 参数匹配情况下，HTTP 应答代码为 302，交互包如下：

![](/assets/密码重置123.png)



可以此特征枚举有效用户名及其邮箱。现在考虑如何制作邮箱字典？很多用户喜欢用用户名注册 qq 邮箱，换言之，用户名 yangyangwithgnu 可能对应邮箱 yangyangwithgnu@qq.com。所以，用前面已经获取有效用户名字典 username.txt 快速制作了邮箱字典 qq-email.txt，其中，username.txt 与 qq-email.txt 逐行对应。

例如，前者第一行为 yangyangwithgnu、后者第一行为 yangyangwithgnu@qq.com。将上面的数据包放入 burp 的 intrduer 中，攻击类型选 pitchfork、user\_name 的参数值定义为枚举变量 1 并加载字典 username.txt、email 的参数值定义为枚举变量 2 并加载字典 qq-email.txt，可枚举出大量有效用户名/邮箱信息，如，zhangfeng/zhangfeng@qq.com、chenchuan/chenchuan@qq.com 等等。

用普通账号 chenchuan/chenchuan@qq.com 演示密码重置漏洞。输入用户名、密码提交，正常完成密码找回逻辑，从交互包中获取服务端下发的重置 token：

```
isVerify
=Y2hlbmNodWFufGNoZW5jaHVhbkBxcS5jb218MTE2MDIzNw==
&
PassPhrase=cbf0160662358808f3586868f041cbaa 
```

拼装为重置链接[http://www.xxxx.com/user/forgotPwdEc.php?isVerify=Y2hlbmNodWFufGNoZW5jaHVhbkBxcS5jb218MTE2MDIzNw==&PassPhrase=cbf0160662358808f3586868f041cbaa](http://www.xxxx.com/user/forgotPwdEc.php?isVerify=Y2hlbmNodWFufGNoZW5jaHVhbkBxcS5jb218MTE2MDIzNw==&PassPhrase=cbf0160662358808f3586868f041cbaa)，访问之，即可进入密码重置页面：

![](/assets/密码重置124.png)

输入新密码 PenTest1024 后系统提示修改成功。用 chenchuan/PenTest1024 成功登录：

  
![](/assets/密码重置125.png)

防御措施上，密码找回的凭证切勿下发客户端，另外，校验邮箱是否有效应添加图片验证码，以防止关键参数被枚举。



