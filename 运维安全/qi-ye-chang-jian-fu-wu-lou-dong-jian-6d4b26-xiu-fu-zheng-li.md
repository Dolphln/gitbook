## 前言 {#_1}

12月份要要给公司同学做安全技术分享，有一块是讲常见服务的漏洞，网上的漏洞检测和修复方案写都比较散，在这里一起做一个整合，整理部分常见服务最近的漏洞和使用上的安全隐患方便有需要的朋友查看。如文章有笔误的地方请与我联系 WeChat:atiger77



## 目录 {#_2}

### 1.内核级别漏洞 {#1}

```
Dirty COW
```

### 2.应用程序漏洞 {#2}

```
Nginx
Tomcat
Glassfish
Gitlab
Mysql
Struts2
ImageMagick
...
```

### 3.应用安全隐患 {#3}

```
SSH
Redis
Jenkins
Zookeeper
Zabbix   
Elasticsearch
Docker
...
```

### 4.总结 {#4}

## 漏洞检测&修复方法 {#_3}

### 1.内核级别漏洞 {#1_1}

**Dirty COW**脏牛漏洞，Linux 内核内存子系统的 COW 机制在处理内存写入时存在竞争，导致只读内存页可能被篡改。

影响范围：Linux kernel &gt;= 2.6.22

漏洞影响：低权限用户可以利用该漏洞写入对自身只读的内存页（包括可写文件系统上对该用户只读的文件）并提权至 root

PoC参考：

* [https://github.com/dirtycow/dirtycow.github.io/wiki/PoCs](https://github.com/dirtycow/dirtycow.github.io/wiki/PoCs)

漏洞详情&修复参考：

* [http://sanwen8.cn/p/53d08S6.html](http://sanwen8.cn/p/53d08S6.html)

* [http://www.freebuf.com/vuls/117331.html](http://www.freebuf.com/vuls/117331.html)

这个漏洞对于使用linux系统的公司来说是一定要修复的，拿web服务举例，我们使用一个低权限用户开放web服务当web被攻击者挂了shell就可以使用exp直接提权到root用户。目前某些云厂商已经在基础镜像中修复了这个问题但是对于之前已创建的主机需要手动修复，具体修复方案可以参考长亭的文章。



### 2.应用程序漏洞 {#2_1}

#### Nginx {#nginx}

Nginx是企业中出现频率最高的服务之一，常用于web或者反代功能。11月15日，国外安全研究员Dawid Golunski公开了一个新的Nginx漏洞（CVE-2016-1247），能够影响基于Debian系列的发行版。

影响范围：

* Debian: Nginx1.6.2-5+deb8u3

* Ubuntu 16.04: Nginx1.10.0-0ubuntu0.16.04.3

* Ubuntu 14.04: Nginx1.4.6-1ubuntu3.6

* Ubuntu 16.10: Nginx1.10.1-0ubuntu1.1

漏洞详情&修复参考：

* [https://www.seebug.org/vuldb/ssvid-92538](https://www.seebug.org/vuldb/ssvid-92538)

这个漏洞需要获取主机操作权限，攻击者可通过软链接任意文件来替换日志文件，从而实现提权以获取服务器的root权限。对于企业来说如果nginx部署在Ubuntu或者Debian上需要查看发行版本是否存在问题即使打上补丁即可，对于RedHat类的发行版则不需要任何修复。



#### Tomcat {#tomcat}

Tomcat于10月1日曝出本地提权漏洞CVE-2016-1240。仅需Tomcat用户低权限，攻击者就能利用该漏洞获取到系统的ROOT权限。

影响范围：

* Tomcat 8 &lt;= 8.0.36-2

* Tomcat 7 &lt;= 7.0.70-2

* Tomcat 6 &lt;= 6.0.45+dfsg-1~deb8u1

受影响的系统包括Debian、Ubuntu，其他使用相应deb包的系统也可能受到影响

漏洞详情&修复参考：

* [http://www.freebuf.com/vuls/115862.html](http://www.freebuf.com/vuls/115862.html)

CVE-2016-4438这一漏洞其问题出在Tomcat的deb包中,使 deb包安装的Tomcat程序会自动为管理员安装一个启动脚本：/etc/init.d/tocat\* 利用该脚本，可导致攻击者通过低权限的Tomcat用户获得系统root权限。

实现这个漏洞必须要重启tomcat服务，作为企业做好服务器登录的权限控制，升级有风险的服务可避免问题。

当然在企业中存在不少部署问题而导致了Tomcat存在安全隐患，运维部署完环境后交付给开发同学，如果没有删除Tomcat默认的文件夹就开放到了公网，攻击者可以通过部署WAR包的方式来获取机器权限。



#### Glassfish {#glassfish}

Glassfish是用于构建 Java EE 5应用服务器的开源开发项目的名称。它基于 Sun Microsystems 提供的 Sun Java System Application Server PE 9 的源代码以及 Oracle 贡献的 TopLink 持久性代码。低版本存在任何文件读取漏洞。

影响范围：Glassfish4.0至4.1

修复参考：升级至4.11或以上版本

PoC参考：

```
http://1.2.3.4:4848/theme/META-INF/%c0.%c0./%c0.%c0./%c0.%c0./%c0.%c0./%c0.%c0./domains/domain1/config/admin-keyfile
```

因为公司有用到Glassfish服务，当时在乌云上看到PoC也测试了下4.0的确存在任何文件读取问题，修复方法也是升级到4.11及以上版本。

#### Gitlab {#gitlab}

Gitlab是一个用于仓库管理系统的开源项目。含义使用Git作为代码管理工具，越来越多的公司从SVN逐步移到Gitlab上来，由于存放着公司代码，数据安全也变得格外重要。

影响范围：

* 任意文件读取漏洞\(CVE-2016-9086\): GitLab CE/EEversions 8.9, 8.10, 8.11, 8.12, and 8.13

* 任意用户authentication\_token泄露漏洞： Gitlab CE/EE versions 8.10.3-8.10.5

漏洞详情&修复参考：

* [http://blog.knownsec.com/2016/11/gitlab-file-read-vulnerability-cve-2016-9086-and-access-all-user-authentication-token/](http://blog.knownsec.com/2016/11/gitlab-file-read-vulnerability-cve-2016-9086-and-access-all-user-authentication-token/)

互联网上有不少公司的代码仓库公网可直接访问，有些是历史原因有些是没有考虑到安全隐患，对于已经部署在公网的情况，可以让Gitlab强制开启二次认证防止暴力破解这里建议使用Google的身份验证，修改默认访问端口，做好acl只允许指定IP进行访问。

####  Mysql

Mysql是常见的关系型数据库之一，翻了下最新的漏洞情况有CVE-2016-6662和一个Mysql代码执行漏洞。由于这两个漏洞实现均需要获取到服务器权限，这里就不展开介绍漏洞有兴趣的可以看下相关文章，主要讲一下Mysql安全加固。

漏洞详情&修复参考：

* [http://bobao.360.cn/learning/detail/3026.html](http://bobao.360.cn/learning/detail/3026.html)

* [http://bobao.360.cn/learning/detail/3025.html](http://bobao.360.cn/learning/detail/3025.html)

在互联网企业中Mysql是很常见的服务，我这边提几点Mysql的安全加固，首先对于某些高级别的后台比如运营，用户等能涉及到用户信息的可以做蜜罐表。在项目申请资源的时候就要做好权限的划分，我们是运维同学保留最高权限，给开发一个只读用户和一个开发权限的用户进行使用，密码一律32位，同时指定机器登录数据库，删除默认数据库和数据库用户。

找了一篇还不错的加固文章提供参考：

* [http://www.it165.net/database/html/201210/3132.html](http://www.it165.net/database/html/201210/3132.html)
* 
#### Struts2 {#struts2}

Struts2是一个优雅的,可扩展的框架,用于创建企业准备的Java Web应用程序。出现的漏洞也着实的多每爆一个各大漏洞平台上就会被刷屏。

漏洞详情&修复参考：

* [http://blog.nsfocus.net/apache-struts2-vulnerability-technical-analysis-protection-scheme-s2-033/](http://blog.nsfocus.net/apache-struts2-vulnerability-technical-analysis-protection-scheme-s2-033/)

* [http://blog.nsfocus.net/apache-struts2-vulnerability-technical-analysis-protection-scheme-s2-037/](http://blog.nsfocus.net/apache-struts2-vulnerability-technical-analysis-protection-scheme-s2-037/)

在线检测平台：

* [http://0day.websaas.com.cn/](http://0day.websaas.com.cn/)

记得有一段时间Struts2的漏洞连续被爆出，自动化的工具也越来越多S2-032，S2-033,S2-037,乌云首页上都是Struts2的漏洞，有国企行业的有证券公司的使用者都分分中招，如果有使用的话还是建议升级到最新稳定版。



#### ImageMagick {#imagemagick}

ImageMagick是一个图象处理软件。它可以编辑、显示包括JPEG、TIFF、PNM、PNG、GIF和Photo CD在内的绝大多数当今最流行的图象格式。

影响范围：

* ImageMagick 6.5.7-8 2012-08-17

* ImageMagick 6.7.7-10 2014-03-06

* 低版本至6.9.3-9released 2016-04-30

漏洞详情&修复参考：

* [http://www.freebuf.com/vuls/104048.html](http://www.freebuf.com/vuls/104048.html)

* [http://www.ithome.com/html/it/223125.htm](http://www.ithome.com/html/it/223125.htm)

这个漏洞爆出来时也是被刷屏的，各大互联网公司都纷纷中招，利用一张构造的图片使用管道服符让其执行反弹shell拿到服务器权限，产生原因是因为字符过滤不严谨所导致的执行代码.对于文件名传递给后端的命令过滤不足,导致允许多种文件格式转换过程中远程执行代码。



### 3.应用安全隐患 {#3_1}

_**为了不加长篇幅长度，加固具体步骤可以自行搜索。**_

**SSH**

之前有人做过实验把一台刚初始化好的机器放公网上看多久会遭受到攻击，结果半个小时就有IP开始爆破SSH的密码，网上通过SSH弱密码进服务器的案列也比比皆是。

**安全隐患：**

```
弱密码
```

**加固建议：**

```
 禁止使用密码登录，更改为使用KEY登录
 禁止root用户登录，通过普通权限通过连接后sudo到root用户
 修改默认端口（默认端口为22）
```



**Redis**

Redis默认是没有密码的，在不需要密码访问的情况下是非常危险的一件事，攻击者在未授权访问 Redis 的情况下可以利用 Redis 的相关方法，可以成功在 Redis 服务器上写入公钥，进而可以使用对应私钥直接登录目标服务器。

安全隐患：

```
未认证访问
开放公网访问
```

加固建议：

```
 禁止把Redis直接暴露在公网
 添加认证，访问服务必须使用密码
```

**Jenkins**

Jenkins在公司中出现的频率也特别频繁，从集成测试到自动部署都可以使用Jenkins来完成，默认情况下Jenkins面板中用户可以选择执行脚本界面来操作一些系统层命令，攻击者通过暴力破解用户密码进脚本执行界面从而获取服务器权限。

安全隐患：

```
登录未设置密码或密码过于简单
开放公网访问
```

加固建议：

```
 禁止把Jenkins直接暴露在公网
 添加认证，建议使用用户矩阵或者与JIRA打通，JIRA设置密码复杂度
```

**Zookeeper**

分布式的，开放源码的分布式应用程序协调服务；提供功能包括：配置维护、域名服务、分布式同步、组服务等。Zookeeper默认也是未授权就可以访问了，特别对于公网开放的Zookeeper来说，这也导致了信息泄露的存在。

安全隐患：

```
开放公网访问
未认证访问
```

加固建议：

```
 禁止把Zookeeper直接暴露在公网
 添加访问控制，根据情况选择对应方式（认证用户，用户名密码，指定IP）
```

  
**Zabbix**

Zabbix为运维使用的监控系统，可以对服务器各项指标做出监控报警，默认有一个不需要密码访问的用户（Guest）。可以通过手工SQL注入获取管理员用户名和密码甚至拿到session，一旦攻击者获取Zabbix登录权限，那么后果不堪设想。

安全隐患：

```
开放公网访问
未删除默认用户
弱密码
```

加固建议：

```
 禁止把Zabbix直接暴露在公网
 删除默认用户
 加强密码复杂度
```

**Elasticsearch**

Elasticsearch是一个基于Lucene的搜索服务器。越来越多的公司使用ELK作为日志分析，Elasticsearch在低版本中存在漏洞可命令执行，通常安装后大家都会安装elasticsearch-head方便管理索引，由于默认是没有访问控制导致会出现安全隐患。

安全隐患：

```
开放公网访问
未认证访问
低版本漏洞
```

加固建议：

```
 禁止把Zabbix直接暴露在公网
 删除默认用户
 升级至最新稳定版
 安装Shield安全插件
```

**Docker**

容器服务在互联网公司中出现的频率呈直线上升，越来越多的公司使用容器去代替原先的虚拟化技术，之前专门做过Docker安全的分析，从 Docker自身安全， DockerImages安全和Docker使用安全隐患进行展开，链接：[https://toutiao.io/posts/2y9xx8/preview](https://toutiao.io/posts/2y9xx8/preview)

之前看到一个外国哥们使用脏牛漏洞在容器中运行EXP跳出容器的视频，具体我还没有复现，如果有复现出来的大家一起交流下~

安全隐患：

```
Base镜像漏洞
部署配置不当
```

加固建议：

```
手动升级Base镜像打上对应补丁
配置Swarm要当心
```

  
4.总结

当公司没有负责安全的同学，做到以下几点可以在一定程度上做到防护：

1. 关注最新漏洞情况，选择性的进行修复；
2. 梳理内部开放服务，了解哪些对外开放能内网访问的绝不开放公网；
3. 开放公网的服务必须做好访问控制；
4. 避免弱密码；避免弱密码；避免弱密码；

以上内容只是理想状态，实际情况即使有安全部门以上内容也不一定能全部做到，业务的快速迭代，开发安全意识的各不相同，跨部门沟通上出现问题等等都会导致出现问题，这篇文章只罗列了部分服务，还有很多服务也有同样的问题，我有空会不断的更新。 

