# 0x00前言

kippo是一款优秀的SSH服务蜜罐，它提供了非常逼真的shell交互环境，比如支持一个文件系统目录的完全伪造，允许攻击者能够增加或者删除其中的文件。Kippo包含一些伪装的文件内容，如/etc/passwd和/etc/shadow等攻击者感兴趣的文件，以UML兼容格式来记录shell会话日志，并提供了辅助工具能够逼真地还原攻击过程。Kippo还引入了很多欺骗和愚弄攻击者的智能响应机制。正是由于具有这些特性，Kippo能够称为是一种中等交互级别的SSH蜜罐软件。

# 0x01 安装

环境依赖：

* Python 2.5+

* Twisted 8.0 to 15.1.0

* PyCrypto

* Zope Interface

安装依赖包：

github的地址：[https://github.com/desaster/kippo](https://github.com/desaster/kippo)

```
cd /home/
wget https://codeload.github.com/desaster/kippo/zip/master
unzip master
yum install twisted python-zope-interface python-pyasn1
yum install python2-paramiko
pip install twisted==15.1.0    (测试过17.1.0-18.7.0都会出现类似的错误：exceptions.ImportError: No module named kippo.core.config)
```

twisted也许安装不了，可以在官网下载安装

[https://twistedmatrix.com/trac/wiki/Downloads](https://twistedmatrix.com/trac/wiki/Downloads)

为了安全，蜜罐是不能以root运行的，需要新创建专用的账户kippo

```
mv kippo-master kippo
useradd kippo
chown -R kippo:kippo
cd kippo
cp kippo.cfg.dist kippo.cfg
```

因为普通账户无法启动1024以下的端口。首先修改一下本机SSH的端口为10086（随意，这是真正的ssh端口）。kippo会监听本地的2222端口，加一条防火墙规则,把22端口转到2222。使用iptables转发，

```
iptables -t nat -A PREROUTING -p tcp --dport 22 -j REDIRECT --to-ports 2222
```

iptables设置：

```
iptables -F   清空iptables设置（慎用）
iptables -t nat -A PREROUTING -p tcp --dport 22 -j REDIRECT --to-ports 2222 添加转发记录
iptables -L -t nat   查看转发规则
```

![](/assets/kippo-nat.png)

启动kippo

```
./start.sh
```

```
[root@localhost kippo]$ ./kippo/start.sh
twistd (the Twisted daemon) 8.2.0
Copyright (c) 2001-2008 Twisted Matrix Laboratories.
See LICENSE for details.
Starting kippo in the background...
/usr/lib64/python2.6/site-packages/twisted/conch/ssh/keys.py:13: DeprecationWarning: the sha module is deprecated; use the hashlib module instead
import sha, md5
/usr/lib64/python2.6/site-packages/twisted/conch/ssh/keys.py:13: DeprecationWarning: the md5 module is deprecated; use hashlib instead
import sha, md5
Generating new RSA keypair...
Done.
Generating new DSA keypair...
Done.
```

使用其他机器扫描一下22端口。

```
lol@huangjianbin:~$ nmap -p 22 10.3.200.11

Starting Nmap 7.40 ( https://nmap.org ) at 2018-08-11 16:39 CST
Nmap scan report for 10.3.200.11
Host is up (0.00062s latency).
PORT   STATE SERVICE
22/tcp open  ssh

Nmap done: 1 IP address (1 host up) scanned in 6.60 seconds
```

可以看到22打开

使用默认账户密码root/123456登陆。

```
root@svr03:/# cat /etc/passwd
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/bin/sh
bin:x:2:2:bin:/bin:/bin/sh
sys:x:3:3:sys:/dev:/bin/sh
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/bin/sh
man:x:6:12:man:/var/cache/man:/bin/sh
lp:x:7:7:lp:/var/spool/lpd:/bin/sh
mail:x:8:8:mail:/var/mail:/bin/sh
news:x:9:9:news:/var/spool/news:/bin/sh
uucp:x:10:10:uucp:/var/spool/uucp:/bin/sh
proxy:x:13:13:proxy:/bin:/bin/sh
www-data:x:33:33:www-data:/var/www:/bin/sh
backup:x:34:34:backup:/var/backups:/bin/sh
list:x:38:38:Mailing List Manager:/var/list:/bin/sh
irc:x:39:39:ircd:/var/run/ircd:/bin/sh
gnats:x:41:41:Gnats Bug-Reporting System (admin):/var/lib/gnats:/bin/sh
nobody:x:65534:65534:nobody:/nonexistent:/bin/sh
libuuid:x:100:101::/var/lib/libuuid:/bin/sh
sshd:x:101:65534::/var/run/sshd:/usr/sbin/nologin
richard:x:1000:1000:Richard Texas,,,:/home/richard:/bin/bash
```

基本黑客一般运行的命令、查看的文件都模拟出来了。

# 0x02 日志存储到数据库

我本人对这一部分暂时没实践，后续会直接将日志进行解析然后存到soc的日志存储中心进行告警。

```
yum install mysql mysql-server
/etc/init.d/mysqld start
mysql -uroot password hehe123
CREATE USER 'kippo'@'localhost' IDENTIFIED BY 'kippo';
create database kippo;
GRANT ALL ON kippo.* to 'kippo'@'localhost' identified by 'kippo';
flush privileges;
```

修改配置文件kippo.cfg

```
[database_mysql]
host = localhost
database = kippo
username = kippo
password = kippo
port = 3306
```

然后导入表结构

```
mysql -ukippo -p -Dkippo < /tmp/kippo/doc/sql/mysql.sql
```

安装python-mysql

```
yum -y install python-devel mysql-devel
wget http://pypi.python.org/packages/source/s/setuptools/setuptools-0.6c11.tar.gz --no-check-certificate
tar -zxvf setuptools-0.6c11.tar.gz
cd setuptools-0.6c11
python setup.py install
wget https://pypi.python.org/packages/source/M/MySQL-python/MySQL-python-1.2.5.zip --no-check-certificate
Unzip MySQL-python-1.2.5.zip
Cd MySQL-python
```

修改site.cfg的mysql\_config一行取消注释

```
mysql_config = /usr/lib64/mysql/mysql_config
python setup.py install
```

来看一下表结构：

```
mysql> show tables;
+-----------------+
| Tables_in_kippo |
+-----------------+
| auth |
| clients |
| downloads |
| input |
| sensors |
| sessions |
+-----------------+
7 rows in set (0.01 sec)
```

执行过的命令：

```
mysql> select * from input;
+----+----------------------------------+---------------------+-------+---------+----------+
| id | session | timestamp | realm | success | input |
+----+----------------------------------+---------------------+-------+---------+----------+
| 1 | 529661ac3e1511e6b417000c292b5908 | 2016-06-29 16:20:15 | NULL | 1 | whoami |
| 2 | 529661ac3e1511e6b417000c292b5908 | 2016-06-29 16:31:10 | NULL | 1 | ifconfig |
+----+----------------------------------+---------------------+-------+---------+----------+
2 rows in set (0.01 sec)
```

连接过的IP：

```
mysql> select * from sessions;
+----------------------------------+---------------------+---------+--------+----------------+----------+--------+
| id | starttime | endtime | sensor | ip | termsize | client |
+----------------------------------+---------------------+---------+--------+----------------+----------+--------+
| 529661ac3e1511e6b417000c292b5908 | 2016-06-29 16:19:58 | NULL | 1 | 172.16.100.128 | 131x25 | 1 |
+----------------------------------+---------------------+---------+--------+----------------+----------+--------+
1 row in set (0.00 sec)
```

# 0x03 文件目录结构

* log：存放日志文件

  ```
   攻击者命令执行格式：
  ```

```
2018-08-11 17:12:58+0800 [SSHChannel session (0) on SSHService ssh-connection on HoneyPotTransport,7,10.3.211.14] CMD: uname -a
2018-08-11 17:12:58+0800 [SSHChannel session (0) on SSHService ssh-connection on HoneyPotTransport,7,10.3.211.14] Command found: uname -a
2018-08-11 17:13:23+0800 [SSHChannel session (0) on SSHService ssh-connection on HoneyPotTransport,7,10.3.211.14] CMD: nc
2018-08-11 17:13:23+0800 [SSHChannel session (0) on SSHService ssh-connection on HoneyPotTransport,7,10.3.211.14] Command not found: nc
2018-08-11 17:13:28+0800 [SSHChannel session (0) on SSHService ssh-connection on HoneyPotTransport,7,10.3.211.14] CMD: yum
```

```
  nmap扫描日志格式：
```

```
2018-08-11 17:22:05+0800 [HoneyPotTransport,9,10.3.208.46] connection lost
```

* txtcmds：存放命令，这些命令都是文本文件，执行相关命令的时候直接显示文件内容
* kippo：核心文件，模拟一些交互式的命令，等等
* data：存放ssh key,lastlog.txt和userdb.txt lastlog.txt:last命令的输出,即存储了登陆蜜罐的信息,也可以伪造 userdb.txt:可以登陆的用户,可以给一个用户设置多个密码,一个用户一行 格式为username:uid:password

  ```
   也会存储一些连接日志，如data/lastlog.txt，有可能是nmap扫描日志，也有可能是真是连接日志
  ```

```
root    pts/0   10.3.208.46     Sat Aug 11 16:46 - 16:46 (00:00)
root    pts/0   10.3.208.46     Sat Aug 11 17:22 - 17:22 (00:00)
```

* honeyfs：etc目录中存在group hostname hosts issue passwd resolv.conf shadow这些 文件,cat /etc/filename目录中对应的文件时会显示这些文本文件中的内容. proc目录中存在cpuinfo meminfo version这些文件,cat /proc/filename目录中对应的文件时会显示这些文本文件中的内容.

* dl: wget等等下载的文件存放的地方

# 0x04 注意点及优缺点

```
1.默认的ssh登录账户名密码为root/12345,也可以进行配置，配置的路径在 /opt/kippo/data/userdb.txt，同时当攻击者登陆后，使用useradd和passwd添加账户和密码的时候也会保存在这个文件里面．

2.当攻击者下载文件时，下载的文件会被保存在/opt/kippo/dl

3.可以设置蜜罐中命令执行后的返回值，也可以添加命令．例如来修改ifconfig命令的输出，可以编辑/opt/kippo/txtcmds/sbin/ifconfig这个文本文件。

4.有一个特点就是，当攻击者输入exit想退出的时候，其实没有退出，只是显示退出，给攻击者一个假象，以为回到的他的本机，他接下来的操作还是会被记录到日志中。

5.你可以更改/opt/kippo/honeyfs，里面保存模拟的系统的文件等内容，使蜜罐更像真实环境。

6.log存储在/opt/kippo/log/kippo.log，你也可以修改配置，把log存储到数据库中，数据库的表结构在 /opt/kippo/doc/sql目录中。

7.每一次登录成功后的操作都会把日志单独再存储一份，存储的路径在/opt/kippo/log/tty,可以通过在/opt/kippo/utils中的playlog.py脚本，来重现这个操作过程。

8.mac尝试登陆会出现bug
2018-08-11 11:37:09+0800 [HoneyPotTransport,2,10.3.208.46] Disconnecting with error, code 3
        reason: couldn't match all kex parts
```

# 0x05 filebeat写入elasticsearch

```
下载filebeat：
wget https://artifacts.elastic.co/downloads/beats/filebeat/filebeat-6.3.2-x86_64.rpm

安装filebeat
rpm -ivh filebeat-6.3.2-x86_64.rpm


编辑 filebeat.yml
vi /etc/filebeat/filebeat.yml

修改之处
- type: log

  # Change to true to enable this input configuration.
  enabled: true

  # Paths that should be crawled and fetched. Glob based paths.
  paths:
    - /home/kippo/kippo/log/kippo.log
    #- c:\programdata\elasticsearch\logs\*
    
    
setup.template:
  name: "filebeat"
  pattern: "filebeat.template.json"
  
  
index: "filebeat-kippo-%{+yyyy.MM.dd}"
  # Array of hosts to connect to.
  hosts: [":9200"]
# Optional protocol and basic auth credentials.
  #protocol: "https"
  username: "test"
  password: "test"
```

效果![](/assets/kippo-es.png)

参考资料：

[https://netsec.ccert.edu.cn/zhugejw/files/2011/09/Kippo-介绍.pdf](https://netsec.ccert.edu.cn/zhugejw/files/2011/09/Kippo-介绍.pdf)

