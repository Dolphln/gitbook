# 蜜罐

## 0x01  什么是蜜罐

蜜罐是存在漏洞的，暴露在外网或者内网的一个虚假的机器,具有以下这些特征：

* 1.其中重要的一点机器是虚假的，攻击者需要花费时间攻破。在这段时间内，系统管理员能够锁定攻击者同时保护真正的机器。
* 2.能够学习攻击者针对该服务的攻击技巧和利用代码。
* 3.一些蜜罐能够捕获恶意软件，利用代码等等，能够捕获攻击者的0day,同时可以帮助逆向工程师通过分析捕获的恶意软件来提高自身系统的安全性
* 4.在内网中部署的蜜罐可以帮助你发现内网中其他机器可能存在的漏洞。
* 蜜罐是把双刃剑，如果不能正确的使用，有可能遭受更多的攻击，模拟服务的软件存在问题，也会产生新的漏洞。

蜜罐分为几下几类：

* １．低交互式：低交互式模拟常规的服务，服务存在漏洞，但是模拟的这些漏洞无法被利用，开发和维护这种类型的蜜罐比较容易。
* ２．高交互式：高交互式使用的是真实的服务，有助于发现服务存在的新漏洞，同时能够记录所有的攻击，但是，部署困难、维护成本高，一旦服务上存在的漏洞被利用，容易引发新的安全问题。
* ３．粘性蜜罐\(Tarpits\)：这种类型的蜜罐，使用新的IP来生成新的虚拟机，模拟存在服务的漏洞，来做诱饵。因此攻击者会花费长时间来攻击，就有足够的时间来处理攻击，同时锁定攻击者。

还有其他类型的蜜罐，比如专门捕获恶意软件的，数据库漏洞利用程序和垃圾邮件等等。当部署两个或者两个以上蜜罐时可以称之为蜜网。

网上关于蜜罐的一些定义：

1.什么是蜜罐：

[http://www.sans.org/security-resources/idfaq/honeypot3.php](http://www.sans.org/security-resources/idfaq/honeypot3.php)

2.蜜网：

[http://www.honeynet.org/](http://www.honeynet.org/)

3.蜜罐项目：

[https://www.projecthoneypot.org/，攻击者的ＩＰ和攻击者的一些数据统计。](https://www.projecthoneypot.org/，攻击者的ＩＰ和攻击者的一些数据统计。)

4.蜜罐的wiki：

[http://en.wikipedia.org/wiki/Honeypot\_\(computing\](http://en.wikipedia.org/wiki/Honeypot_%28computing\)\)

## 0x02 MHN

MHN是一个开源软件，它简化了蜜罐的部署，同时便于收集和统计蜜罐的数据。用ThreatStream（[http://threatstream.github.io/mhn/）来部署，MHN使用开源蜜罐来收集数据，整理后保存在Mongodb中，收集到的信息也可以通过web接口来展示或者通过开发的API访问。](http://threatstream.github.io/mhn/）来部署，MHN使用开源蜜罐来收集数据，整理后保存在Mongodb中，收集到的信息也可以通过web接口来展示或者通过开发的API访问。)

MHN能够提供多种开源的蜜罐，可以通过web接口来添加他们。一个蜜罐的部署过程很简单，只需要粘贴，复制一些命令就可以完成部署，部署完成后，可以通过开源的协议hpfeeds来收集的信息。

MHN支持以下蜜罐：

* 1.Snort:[https://www.snort.org/](https://www.snort.org/)                IDS
* 1. Suricata:[http://suricata-ids.org/](http://suricata-ids.org/)          IDS
* 1. Dionaea:[http://dionaea.carnivore.it/,它是一个低交互式的蜜罐，能够模拟MSSQL](http://dionaea.carnivore.it/,它是一个低交互式的蜜罐，能够模拟MSSQL), SIP, HTTP, FTP, TFTP等服务 drops中有一篇介绍：[http://drops.wooyun.org/papers/4584](http://drops.wooyun.org/papers/4584)
* 1. Conpot:[http://conpot.org/](http://conpot.org/)     低交互工业控制蜜罐
* 1. Kippo:[https://github.com/desaster/kippo，它是一个中等交互的蜜罐，能够下载任意文件。](https://github.com/desaster/kippo，它是一个中等交互的蜜罐，能够下载任意文件。) drops中有一篇介绍：[http://drops.wooyun.org/papers/4578](http://drops.wooyun.org/papers/4578)
* 1. Amun:[http://amunhoney.sourceforge.net/，它是一个低交互式蜜罐，但是已经从2012年之后不在维护了。](http://amunhoney.sourceforge.net/，它是一个低交互式蜜罐，但是已经从2012年之后不在维护了。)
* 1. Glastopf：[http://glastopf.org/](http://glastopf.org/)
* 1. Wordpot：[https://github.com/gbrindisi/wordpot](https://github.com/gbrindisi/wordpot)
* 1. ShockPot：[https://github.com/threatstream/shockpot，模拟的CVE-2014-6271，即破壳漏洞](https://github.com/threatstream/shockpot，模拟的CVE-2014-6271，即破壳漏洞)
* 1. p0f：[https://github.com/p0f/p0f](https://github.com/p0f/p0f)
* 11.cowrie: [https://github.com/cowrie/cowrie](https://github.com/cowrie/cowrie)     类似kippo的ssh服务蜜罐

