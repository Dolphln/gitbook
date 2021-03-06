# 业务、web安全

**在逻辑漏洞中，任意用户密码重置最为常见，可能出现在新用户注册页面，也可能是用户登录后重置密码的页面，或者用户忘记密码时的密码找回页面，其中，密码找回功能是重灾区。我把日常渗透过程中遇到的案例作了漏洞成因分析，这次，关注因重置凭证接收端可篡改导致的任意用户密码重置问题。**

密码找回逻辑含有用户标识（用户名、用户 ID、cookie）、接收端（手机、邮箱）、凭证（验证码、token）、当前步骤等四个要素，若这几个要素没有完整关联，则可能导致任意密码重置漏洞。

**前情提要：【**[**传送门**](http://www.freebuf.com/articles/web/160883.html)**】**

## **案例一：接收端可篡改。请求包中包含接收端参数，可将凭证发至指定接收端。**

密码重置页面，输入任意普通账号，选择手机方式找回密码。在身份验证页面点击获取短信验证码：  
![](https://github.com/Dolphln/gitbook/tree/6c5fd24eb08f8313b5976415a1da555bc6005b73/assets/密码重置21.png)

拦截请求，发现接收验证码的手机号为请求包中的参数：![](https://github.com/Dolphln/gitbook/tree/6c5fd24eb08f8313b5976415a1da555bc6005b73/assets/密码重置22.png)直接篡改为攻击者的手机号，成功接收短信验证码，提交验证码后，正常执行 3、4 步即可成功重置该账号的密码。

## **案例二：接收端可篡改。请求包中出现接收端间接相关参数，可将凭证发至指定接收端。**

在密码找回页面，用攻击账号 test0141，尝试重置目标账号 2803870097 的密码（对滴，你没看错，这两个长得完全不像的账号的确是同个网站的）。

在第一个首页中输入 test0141 和图片验证码完成“01 安全认证”：

![](https://github.com/Dolphln/gitbook/tree/6c5fd24eb08f8313b5976415a1da555bc6005b73/assets/密码重置23.png!small)

请求为：

![](https://github.com/Dolphln/gitbook/tree/6c5fd24eb08f8313b5976415a1da555bc6005b73/assets/密码重置24.png!small)

输入图片验证码获取短信验证码完成“02 身份验证”：

![](https://github.com/Dolphln/gitbook/tree/6c5fd24eb08f8313b5976415a1da555bc6005b73/assets/密码重置25.png!small)

请求为：

![](https://github.com/Dolphln/gitbook/tree/6c5fd24eb08f8313b5976415a1da555bc6005b73/assets/密码重置26.png!small)

后续的 03、04 步不涉及用户名信息，忽略。

全流程下来，客户端并未直接提交接收短信验证码的手机号，多次尝试可知，02 中出现的 user\_name 用于查询下发短信的手机号，用它可以间接指定接收端，那么，它是否仅此作用而不用于指定重置密码的账号？如下思路验证，先将 userName 置为 2803870097 完成 01 以告诉服务端重置的账号，再将 user\_name 置为 test0141 完成 02 以欺骗服务端将短信验证码发至攻击者手机，顺序完成 03、04 或许能实现重置 2803870097 的密码。具体如下。

第一步，用普通账号 2803870097 进行安全认证：

![](https://github.com/Dolphln/gitbook/tree/6c5fd24eb08f8313b5976415a1da555bc6005b73/assets/密码重置27.png!small)

第二步，对普通账号 2803870097 进行身份验证：

![](https://github.com/Dolphln/gitbook/tree/6c5fd24eb08f8313b5976415a1da555bc6005b73/assets/密码重置28.png!small)

拦截发送短信验证码的请求：

![](https://github.com/Dolphln/gitbook/tree/6c5fd24eb08f8313b5976415a1da555bc6005b73/assets/密码重置29.png!small)

将 user\_name 从 2803870097 篡改为 test0141，控制服务端将验证码发至 test0141 绑定的手机号：

![](https://github.com/Dolphln/gitbook/tree/6c5fd24eb08f8313b5976415a1da555bc6005b73/assets/密码重置210.png)

test0141 的手机号成功接收到验证码 872502，将该验证码填入重置 2803870097 的身份校验页面后提交：

![](https://github.com/Dolphln/gitbook/tree/6c5fd24eb08f8313b5976415a1da555bc6005b73/assets/密码重置211.png!small)

第三步，输入新密码 PenTest1024 后提交，系统提示重置成功：

![](https://github.com/Dolphln/gitbook/tree/6c5fd24eb08f8313b5976415a1da555bc6005b73/assets/密码重置212.png!small)

第四步，用 2803870097/PenTest1024 登录，验证成功：

防御措施方面，一定要将重置用户与接收重置凭证的手机号/邮箱作一致性比较，通常直接从服务端直接生成手机号/邮箱，不从客户端获取。

