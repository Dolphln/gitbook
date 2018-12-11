Egor Homakov（Twitter: [@homakov](http://twitter.com/homakov)个人网站: [EgorHomakov.com](http://egorhomakov.com/)）是一个Web安全的布道士，他这两天把github给黑了，并给github报了5个安全方面的bug，他在他的这篇blog——《[How I hacked Github again](http://homakov.blogspot.com/2014/02/how-i-hacked-github-again.html)》（墙）说明了这5个安全bug以及他把github黑掉的思路。Egor的这篇文章讲得比较简单，很多地方一笔带过，所以，**我在这里用我的语言给大家阐述一下黑掉Github的思路以及原文中所提到的那5个bug。希望这篇文章能让从事Web开发的同学们警惕**。关于Web开发中的安全事项，大家可以看看这篇文章《[Web开发中的你需要了解的东西](https://coolshell.cn/articles/6043.html)》

![](/assets/git-1.png)

#### OAuth简介

首先，这个故事要从[Github OAuth](https://developer.github.com/v3/oauth/)讲起。所以，我们需要先知道什么是[OAuth](http://en.wikipedia.org/wiki/OAuth)。所谓OAuth就是说，第三方的应用可以通过你的授权而不用知道你的帐号密码能够访问你在某网站的你自己的数据或功能。像Google, Facebook, Twitter等网站都提供了OAuth服务，提供OAuth服务的网站一般都有很多开放的API，第三方应用会调用这些API来开发他们的应用以让用户拥有更多的功能，但是，当用户在使用这些第三方应用的时候，这些第三方的应用会来访问用户的帐户内的功能和数据，所以，当第三应用要干这些事的时候，我们不能让第三方应用弹出一个对话框来问用户要他的帐号密码，不然第三方的应用就把用户的密码给获取了，所以，OAuth协议会跳转到一个页面，让用户授权给这个第三方应用以某些权限，然后，这个权限授权的记录保存在Google/Facebook/Twitter上，并向第三方应用返回一个授权token，于是第三方的应用通过这个token来操作某用户帐号的功能和数据时，就畅通无阻了。下图简单地说明了Twitter的OAuth的授权过程。

![](/assets/git-2.png)





