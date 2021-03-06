# 运维安全

因为在全网段资产扫描时，nmap已经无法满足使用。无论添加-n -sS -T5 -Pn 等。不得已转用最快速度的masscan。masscan 也能够自定义扫描对象，甚至排除端口、排除特定IP，不同IP之间用英文状态逗号连接等。

扫描指定主机的特定端口：

```text
masscan.exe -p80,443 192.168.81.143
```

扫描ip段特定端口：

```text
masscan 10.11.0.0/16 -p80,443,8080 --rate 1000000
```

获取banner：

```text
masscan.exe -p80,443,3306 192.168.81.143 --banners
```

通过配置文件启动扫描：

```text
将配置信息保存在1.conf:
```

```text
masscan.exe -p80,443,3306 192.168.81.143 --banners --echo>1.conf
```

读取配置信息1.conf，启动扫描:

```text
masscan.exe -c 1.conf
```

修改扫描速度为100,000包/秒（Windos下最大为 300,000包/秒），默认100包/秒：

```text
--rate 100000
```

输出格式：

```text
-oX <filespec>(XML)
-oB <filespec>(Binary)
-oG <filespec>(Grep)
-oJ <filespec>(Json)
-oL <filespec>(List)
-oU <filespec>(Unicornscan format)
```

补充，默认情况，masscan开启如下配置：

```text
-sS: this does SYN scan only (currently, will change in the future) 
-Pn: doesn't ping hosts first, which is fundamental to the async operation 
-n: no DNS resolution happens 
--randomize-hosts: scan completely randomized 
--send-eth: sends using raw libpcap
```

详细参数

```text
-p 《ports,–ports 《ports》》 指定端口进行扫描 
–banners 获取banner信息，支持少量的协议 
–rate 《packets-per-second》 指定发包的速率 
-c 《filename》, –conf 《filename》 读取配置文件进行扫描 
–echo 将当前的配置重定向到一个配置文件中 
-e 《ifname》 , –adapter 《ifname》 指定用来发包的网卡接口名称 
–adapter-ip 《ip-address》 指定发包的IP地址 
–adapter-port 《port》 指定发包的源端口 
–adapter-mac 《mac-address》 指定发包的源MAC地址 
–router-mac 《mac address》 指定网关的MAC地址 
–exclude 《ip/range》 IP地址范围黑名单，防止masscan扫描 
–excludefile 《filename》 指定IP地址范围黑名单文件 
–includefile，-iL 《filename》 读取一个范围列表进行扫描 
–ping 扫描应该包含ICMP回应请求 
–append-output 以附加的形式输出到文件 
–iflist 列出可用的网络接口，然后退出 
–retries 发送重试的次数，以1秒为间隔 
–nmap 打印与nmap兼容的相关信息 
–http-user-agent 《user-agent》 设置user-agent字段的值 
–show [open,close] 告诉要显示的端口状态，默认是显示开放端口 
–noshow [open,close] 禁用端口状态显示 
–pcap 《filename》 将接收到的数据包以libpcap格式存储 
–regress 运行回归测试，测试扫描器是否正常运行 
–ttl 《num》 指定传出数据包的TTL值，默认为255 
–wait 《seconds》 指定发送完包之后的等待时间，默认为10秒 
–offline 没有实际的发包，主要用来测试开销 
-sL 不执行扫描，主要是生成一个随机地址列表 
–readscan 《binary-files》 读取从-oB生成的二进制文件，可以转化为XML或者JSON格式. 
–connection-timeout 《secs》 抓取banners时指定保持TCP连接的最大秒数，默认是30秒。
```

安装参考：

github页面：

`https://github.com/robertdavidgraham/masscan`

Ubuntu的安装过程如下：

```text
$ sudo apt-get install git gcc make libpcap-dev clang
$ git clone https://github.com/robertdavidgraham/masscan
$ cd masscan
$ make
```

