# 缘起\(Why\)

## 现有环境

* KVM
* CentOS 6.x
* OpenVPN 2.3.2
* Google Authenticator libPAM 1.0.1
* pam\_ldap 185
* Windows AD\(2008R2\)

## 来自老板的需求

1. 希望加强登录认证，仅仅靠原来的基于 AD 的认证还不够

## 老板认可的方案

1. 用 Google Authenticator 来做动态的二次认证
2. 结合原有的 ldap 集中认证

# 具体步骤\(Howto\)

## 服务器端

### 制作google-authenticator的rpm包

* 其实 epel 源里有 google-authenticator 的 rpm 包，但是那个太老了\( 0.3?\)，很多参数不支持，没法用。
* 随便一台 CentOS 6.x for x86\_64 的机器上编都可以

```
# 准备打包的软件环境
yum -y install git gcc \
        libtool autoconf \
        automake pam-devel \
        rpm-build qrencode-libs;
# 抓取源代码，打包
git clone \
    https://github.com/google/google-authenticator-libpam.git;
cd google-authenticator-libpam;
./bootstrap.sh;
./configure;
make dist;
cp google-authenticator-*.tar.gz ~/rpmbuild/SOURCES/;
# 上面这一步如果提示~/rpmbuild目录不存在
# 则先把下一步执行一次然后再试
# （下面这一步会确保有~/rpmbuild目录）
rpmbuild -ba contrib/rpm.spec;
# 成功以后，编好的rpm包路径在：
# ~/rpmbuild/RPMS/x86_64/google-authenticator-1.*.el6.x86_64.rpm
```

### 

### 安装软件

在要跑 OpenVPN 服务的目标服务器上，

```
# 安装前一步制作的google-authenticator的rpm包
rpm -ivh google-authenticator-1.*.el6.x86_64.rpm;
yum -y install openvpn \
        pam_ldap \
        openvpn-auth-ldap \
        pamtester;
# 如果是新装的服务器（我这里自然不是），请别忘了装openvpn
# 上面的openvpn-auth-ldap和pamtester都不是必须要装的
# pamtester用来做一些pam认证的测试
```

### 

### 配置软件

#### /etc/pam\_ldap.conf

这是 CentOS 6.x 下 pam\_ldap.so 的配置文件，因为我们后面在 OpenVPN 的 pam 认证里会用 pam\_ldap.so 来从 AD 认证，所以这里我们需要先把这个配置好。

```
# host 定义的是域控的 ip
# 可以多写几个
# 用空格分开即可
# 请根据实际情况填写
host 10.0.0.1
# base 写的是用户的 basedn
# 这个需要根据具体情况修改
base ou=staff,dc=xxx,dc=com
# binddn 是用来搜索 AD 域的帐号
binddn CN=xxx_ldap,OU=access account,OU=staff,DC=xxx,DC=com
# bindpw 是搜索 AD 域的帐号的密码
bindpw passwordxxx
# pam_filter 可以设一些允许连接 OpenVPN 服务的条件
# 我司这里用了一个允许拨入的条件
# 这个不是必须的
pam_filter msNPAllowDialin=TRUE
# pam_login_attribute 是来设定 AD 域上
# 跟 OpenVPN 里登录的用户名匹配的属性
# AD 这里一般都是 sAMAccountName
pam_login_attribute sAMAccountName
# ssl 设置是否使用加密，我司这里没有使用加密
ssl off
```

#### /etc/openvpn/xxx.conf

这是 openvpn 服务器的主配置文件，"xxx" 用自己喜欢的字串替换，这个配置最重要的是以下两句：

```
plugin /usr/lib64/openvpn/plugins/openvpn-plugin-auth-pam.so openvpn reneg-sec 36000
```

这里配置的基本思想就是：

* 把认证丢给系统的 pam 插件来做
* 这里的 openvpn 参数指的是 pam 的配置文件（路径在 /etc/pam.d/）
* reneg-sec 是用来设置重新认证的时间间隔，这里是 10 小时。这个参数跟客户端的同样的参数取小者有效

#### /etc/pam.d/openvpn

这个配置文件是真正认证 openvpn 登录的配置文件，里面内容主要是以下几句：

```
auth    required    pam_google_authenticator.so nullok forward_pass debug
auth    required    pam_ldap.so use_first_pass debug
account required    pam_unix.so
```

* 第一行的 forward\_pass 参数使得一次读入系统密码\(ldap，也就是 AD 密码\)和 google authenticator 的密码，然后把系统密码扔给后续的pam（也就是带有 use\_first\_pass 参数的pam模块）处理

* 第二行的 use\_first\_pass 上面已有讲到

* 前两行都带 debug 参数完全是调试的需要，生产环境可以不用

* 第三行是必须的，否则 openvpn 登录不上

### 

### 启动服务

```
chkconfig openvpn on;
service openvpn start;
```

### 

### 用户初始化

因为 google authenticator 需要在每个用户的家目录里生成一个叫 .google\_authenticator 的文件，里面存有密钥和几个一次性的超级认证码及其他一些配置，所以，我们需要在 openvpn 服务器上做一次初始化数据的工作。假设有 100 个用户要用这个系统登录 openvpn，用户名从 user1 到 user100，那么初始化脚本就是：

```
for i in {1..100}
do
    useradd user${i}
    su -c \
        "google-authenticator -t -f -d -w 17 -r 3 -R 30 -q" \
        user${i}
    cat /home/user${i}/.google_authenticator | \
        mail -s "your .google_authenticator is:" \
        user${i}@xxx.com
done
```

这里注意，在用户数据初始化的最后部分，将生成的 .google\_authenticator 发给了每个用户自己。

## 

## 客户端

### 软件安装及配置使用

#### Google Authenticator

这个软件准确来讲不是必须在 openvpn 客户端上安装的软件，只不过这个软件是用来生成登录所需的 6 位动态数字密码的，一般安装在手机上。ios 系统和 android 系统分别在 app store 和 google play 里搜索 "google authenticator" 然后安装即可。

这个软件的使用也很简单，本地运行，根本不需要“科学上网”。具体以 iphone 版本为例：打开软件，第一次点击右上角“+”（加号）--&gt;手动输入验证码，然后：

* 账号：随便填
* 密钥：.google\_authenticator 文件（内容已经邮件发给个人）的第一行
* 基于时间：划到打开状态

这样以后每次打开软件，都会看到每30秒生成一个新的6位数字，这就是二次认证的验证码

#### OpenVPN客户端

OpenVPN 客户端软件各个平台都有很多，我们在 mac 下常用 tunnelblick，用啥不重要，重要的是要在配置文件里加一句话：

```
reneg-sec 36000
```

这个参数具体配多少随意，真正生效的是和服务器端同样的参数相比的小者，而本例中服务器端配的是 36000。

客户端软件使用时需要把手机放手边打开 Google Authenticator 应用，输入密码时输入系统密码+6位验证码这样的组合，比如，用户密码是 "woshimima"\(不带引号\)，手机上显示的6位验证码是 "123456"（不带引号），那么输入密码时我就要输入 "woshimima123456"（不带引号）

## 

## 简单测试

在 OpenVPN 服务器上，使用前面安装的 pamtester 来测试 OpenVPN 认证，方法很简单，登录 OpenVPN 服务器，执行命令：

```
pamtester openvpn san.zhang authenticate;
# 系统会提示输入密码和 Google Authenticator 的 6 位动态密码
# 一起输入，先输入域帐号 san.zhang 的密码
# 这个测试必须要通过，否则整个配置肯定是失败的
```

# 参考链接

* [google-authenticator@github](https://link.jianshu.com/?t=https://github.com/google/google-authenticator-libpam)
* [google-authenticator的rpm包制作官方文档](https://link.jianshu.com/?t=https://github.com/google/google-authenticator-libpam/blob/master/contrib/README.rpm.md)
  （注意：这个文档有问题，起码在 CentOS 6.x 下是有问题的）



