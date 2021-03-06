# 一个登陆框引起的血案

## 一个登陆框引起的血案

**客户给的测试范围，或者挖众测时，很多时候都只有一个简单的登陆框，想起当初的苦逼的我，只能去测测爆破弱口令，而且还是指定用户名爆破密码这种，当真是苦不堪言；**

文章内容很简单，但是还是想分享一波，送给向我一样的孩子。

## 0×00 附文章内容结构图

[    
](http://image.3001.net/images/20180609/1528517724927.png)![](../.gitbook/assets/deng-lu-kuang-xie-an-1.png!small)

## 0×01 暴力破解

### 1. 指定用户名爆破密码

传统型爆破思路，用户名可以通过猜测或者信息收集获得。

猜测：admin、网站域名等

信息收集：新闻发布人、whoami等

![](../.gitbook/assets/deng-lu-kuang-xie-an-2.png!small)

### 2. 指定密码爆破用户名

如果是后台登陆处，那么性价比会降低，因为后台登陆处，用户名可能会很少，甚至只有一个。

更加适用于普通用户登陆处。

指定弱口令爆破用户名，拿TOP1弱口令123456尝试，百试不爽。

分享一个遇到过的看似比较费劲的防御措施

![](../.gitbook/assets/deng-lu-kuang-xie-an-3.png!small)

编写脚本绕过防御策略

![](../.gitbook/assets/deng-lu-kuang-xie-an-4.png!small)

再分享一次遇到特别恶心的一次，用BurpSuite爆破时，响应包长度、状态码完全相同；

那时候还没有设置关键字匹配数据包的意识，甚是悲催，

我说：没有弱口令；同事：有啊，分明有很多。

在爆破的时候，添加匹配关键字：

可以添加登陆成功时，独有的关键字；

也可以添加登陆失败时，独有的关键字。

![](../.gitbook/assets/deng-lu-kuang-xie-an-5.png!small)

然后返回结果这里，便会发现多出了一列，匹配到关键字的带有对勾，没有匹配到的则空白

![](../.gitbook/assets/deng-lu-kuang-xie-an-6.png!small)

## 0×02 SQL注入

### 1. 万能密码

![](../.gitbook/assets/deng-lu-kuang-xie-an-7.png!small)

![](../.gitbook/assets/deng-lu-kuang-xie-an-8.png!small)

### 2.SQL注入

![](../.gitbook/assets/deng-lu-kuang-xie-an-9.png!small)

## 0×03 Self-XSS+CSRF

经测试发现用户登陆处存在XSS，但只是Self-XSS，自己插自己，不用灰心，再看看这个登录框是否存在CSRF即可。

[    
](http://image.3001.net/images/20180609/15285189248545.png)![](../.gitbook/assets/deng-lu-kuang-xie-an-10.png!small)

![](../.gitbook/assets/deng-lu-kuang-xie-an-11.png!small)

构造CSRF POC，将XSS的payload放到用户名这里。

![](../.gitbook/assets/deng-lu-kuang-xie-an-12.png!small)

测试后，发现成功弹窗

![](../.gitbook/assets/deng-lu-kuang-xie-an-13.png!small)

## 0×04 任意用户注册

如果登陆框附近存在用户注册功能时，可以尝试

### 1. 失效的身份认证

如校验值默认为空

![](../.gitbook/assets/deng-lu-kuang-xie-an-14.png!small)

![](../.gitbook/assets/deng-lu-kuang-xie-an-15.png!small)

### 2. 验证码可暴破

简单粗暴

## 0×05 任意密码重置

任意密码重置姿势太多，附上我之前做的脑图

一些详情，可以移步我的博客

[http://teagle.top/index.php/logic.html](http://teagle.top/index.php/logic.html)

![](../.gitbook/assets/deng-lu-kuang-xie-an-16.png!small)

呃。。赘述一种我比较喜欢的方式，在找回密码处不存在任意密码重置漏洞时，不用灰心，登陆进去，在个人中心处依旧会有很大几率存在任意密码重置漏洞。

如：

CSRF重新绑定手机号、邮箱号，

重新绑定时，用户身份可控，如最后的请求包可以通过修改用户id来控制绑定的用户

## 0×06 短信轰炸

存在用户注册、用户找回密码等功能时，尝试是否存在短信炸弹

### 1. 单个用户短信炸弹

指定单个用户，然后重放发送短信的HTTP请求。

BurpSuite中的一个Tricks：不修改参数，直接重放数据包，对于短信炸弹的测试非常实用

![](../.gitbook/assets/deng-lu-kuang-xie-an-17.png!small)

### 2. 轮询用户

每次测试这个，都是使用学校里的手机卡，遍历后面的几位，这样就可以直接询问同学是否收到短信；

每次都很刺激。

![](../.gitbook/assets/deng-lu-kuang-xie-an-18.png!small)

