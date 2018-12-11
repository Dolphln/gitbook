## 安装

```
$ sudo apt-get install clang git gcc make libpcap-dev
$ git clone https://github.com/robertdavidgraham/masscan
$ cd masscan
$ make
```

## 使用

**单端口扫描**

```
Masscan 10.11.0.0/16 -p443
```

**多端口扫描**

扫描80或443端口的B类子网

```
Masscan 10.11.0.0/16 -p80,443
```

**扫描一系列端口**

扫描22到25端口的B类子网

```
Masscan 10.11.0.0/16 -p22-25
```

**快速扫描**

使用如上的的设置可以得到结果，但速度将是比较慢。正如已经讨论的那样，整体上masscan要快一点，所以让我们加快速度。

默认情况下，Masscan扫描速度为每秒100个数据包，这是相当慢的。为了增加这一点，只需提供该-rate选项并指定一个值。

扫描100个常见端口的B类子网，每秒100,000个数据包

```
Masscan 10.11.0.0/16  --top-ports 100 -rate 100000
```

你可以扫描的速度取决于很多因素，包括您的操作系统（Linux扫描扫描远远快于Windows），系统的资源，最重要的是您的带宽。为了以高速扫描非常大的网络，您需要使用百万以上的速率（-rate 1000000）。

**输出格式**

```
XML 默认格式 使用-oX <filename> 或者使用 –output-format xml 和 –output-filename <filename>进行指定

binary masscan内置格式

grepable nmap格式 使用 -oG <filename> 或者 –output-format grepable 和 –output-filename <filename>进行指定

json 使用 -oJ <filename> 或者 –output-format json 和 –output-filename <filename>进行指定

list 简单的列表,每行一个主机端口对。使用-oL <filename> 或者 –output-format list 和 –output-filename <filename>进行指定
```

**配置文件启动扫描**

将配置信息保存在1.conf:

```
masscan.exe -p80,443,3306 192.168.81.143 --banners --echo>1.conf
```

读取配置信息1.conf，启动扫描:

```
masscan.exe -c 1.conf
```

补充，默认情况，masscan开启如下配置：

```
-sS: this does SYN scan only (currently, will change in the future) 
-Pn: doesn't ping hosts first, which is fundamental to the async operation 
-n: no DNS resolution happens 
--randomize-hosts: scan completely randomized 
--send-eth: sends using raw libpcap
```

**详细参数**

```
详细参数 
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



