参考链接：[https://coolshell.cn/articles/11973.html](https://coolshell.cn/articles/11973.html)



很多人或许对上半年发生的安全问题“心脏流血”（Heartbleed Bug）事件记忆颇深，这两天，又出现了另外一个“毁灭级”的漏洞——Bash软件安全漏洞。这个漏洞由法国GNU/Linux爱好者Stéphane Chazelas所发现。随后，美国电脑紧急应变中心（US-CERT）、红帽以及多家从事安全的公司于周三（北京时间9月24日）发出警告。 关于这个安全漏洞的细节可参看美国政府计算安全的这两个漏洞披露：[CVE-2014-6271](http://web.nvd.nist.gov/view/vuln/detail?vulnId=CVE-2014-6271) 和 [CVE-2014-7169](http://web.nvd.nist.gov/view/vuln/detail?vulnId=CVE-2014-7169)。

这个漏洞其实是非常经典的“注入式攻击”，也就是可以向 bash注入一段命令，从bash1.14 到4.3都存在这样的漏洞。我们先来看一下这个安全问题的症状。

![](/assets/shellsock-1.png)

### Shellshock \(CVE-2014-6271\)

下面是一个简单的测试：

```ruby
$ env VAR='() { :;}; echo Bash is vulnerable!' bash -c "echo Bash Test"
```

如果你发现上面这个命令在你的bash下有第一行的输出，那你就说明你的bash是有漏洞的：

```ruby
Bash is vulnerable!
Bash Test
```

简单地看一下，其实就是向环境变量中注入了一段代码**echo Bash is vulnerable**。关于其中的原理我会在后面给出。

很快，CVE-2014-6271的官方补丁出来的了——[Bash-4.3 Official Patch 25](https://lists.gnu.org/archive/html/bug-bash/2014-09/msg00081.html)。



### AfterShock – CVE-2014-7169 （又叫Incomplete fix to Shellshock）

但随后，马上有人在Twitter上发贴——[说这是一个不完整的fix](http://twitter.com/taviso/statuses/514887394294652929)，并给出了相关的攻击方法。

![](/assets/shellsock-2.png)

也就是下面这段测试代码（注意，其中的sh在linux下等价于bash）：

```ruby
env X='() { (a)=>\' sh -c "echo date"; cat echo
```

上面这段代码运行起来会报错，但是它要的就是报错，报错后会在你在当前目录下生成一个echo的文件，这个文件的内容是一个时间文本。下面是上面 这段命令执行出来的样子。

```cpp
$ env X='() { (a)=>\' sh -c "echo date"; cat echo
sh: X: line 1: syntax error near unexpected token `='
sh: X: line 1: `'
sh: error importing function definition for `X'
Sat Sep 27 22:06:29 CST 2014
```

这段测试脚本代码相当的诡异，就像“天书”一样，我会在后面详细说明这段代码的原理。

  


### 原理和技术细节

要说清楚这个原理和细节，我们需要从 bash的环境变量开始说起。

#### bash的环境变量

环境变量大家知道吧，这个不用我普及了吧。环境变量是操作系统运行shell中的变量，很多程序会通过环境变量改变自己的执行行为。在bash中要定义一个环境变量的语法很简单（注：=号的前后不能有空格）：

```cpp
$ var="hello world"
```

然后你就可以使用这个变量了，比如：echo $var什么的。但是，我们要知道，这个变量只是一个当前shell的“局部变量”，只在当前的shell进程中可以访问，这个shell进程fork出来的进程是访问不到的。

你可以做这样的测试：

```cpp
$ var="hello coolshell"
$ echo $var
hello coolshell
$ bash
$ echo $var
```

上面的测试中，第三个命令执行了一个bash，也就是开了一个bash的子进程，你就会发现var不能访问了。

为了要让shell的子进程可以访问，我们需要export一下：

```cpp
$ export var="hello coolshell"
```

这样，这个环境变量就会在其子进程中可见了。

如果你要查看一下有哪些环境变量可以在子进程中可见（也就是是否被export了），你可使用**env**命令。不过，env命令也可以用来定义export的环境变量。如下所示：

```cpp
$ env var="hello haoel"
```

有了这些基础知识还不够，我们还要知道一个基础知识——shell的函数。

#### bash的函数

在bash下定义一个函数很简单，如下所示：

```cpp
$ foo(){ echo "hello coolshell"; }
$ foo
hello coolshell
```

有了上面的环境变量的基础知识后，你一定会想试试这个函数是否可以在子进程中调用，答案当然是不行的。

```go
$ foo(){ echo "hello coolshell"; }
$ foo
hello coolshell
$ bash
$ foo
bash: foo: command not found
```

你看，和环境变量是一样的，如果要在子进程中可以访问的话，那么，还是一样的，需要export，export有个参数 -f，意思是export一个函数。如：

```ruby
$ foo(){ echo "hello coolshell"; }
$ foo
hello coolshell
$ export -f foo
$ bash
$ foo
hello coolshell
```

好了，我讲了这么半天的基础知识，别烦，懂了这些，你才会很容易地理解这两个漏洞是怎么回事。

好，现在要进入正题。

####  bash的bug

从上面我们可以看到，bash的变量和函数用了一模一样的机制，如果你用env命令看一下export出来的东西，你会看到上面我们定义的变量和函数都在，如下所示（我省略了其它的环境变量）：

```ruby
$ env
var=hello coolshell
foo=() { echo "hello coolshell"
}
```

原来，都用同样的方式啊——**无论是函数还是变量都是变量啊**。于是，看都不用看bash的源代码，聪明的黑客就能猜得到——

**bash判断一个环境变量是不是一个函数，就看它的值是否以”\(\)”开始**。于是，一股邪念涌上心头。



黑客定义了这样的环境变量（注：\(\) 和 { 间的空格不能少）：

```ruby
$ export X='() { echo "inside X"; }; echo "outside X";'
```

env一下，你会看到X已经在了：

```ruby
$ env
X=(){ echo "inside X"; }; echo "outside X";
```

然后，**当我们在当前的bash shell进程下产生一个bash的子进程时，新的子进程会读取父进程的所有export的环境变量，并复制到自己的进程空间中，很明显，上面的X变量的函数的后面还注入了一条命令：echo “outside X”，这会在父进程向子进程复制的过程中被执行吗？**（关于fork相关的东西你可以看一下我以前写的《[fork的一个面试题](https://coolshell.cn/articles/7965.html)》）

答案是肯定的。

```ruby
$ export X='() { echo "inside X"; }; echo "outside X";'
$ bash
outside X
```

你看，一个代码注入就这样完成了。这就是bash的bug——**函数体外面的代码被默认地执行了**。



我们并不一定非要像上面那样创建另一个bash的子进程，我们可以使用bash -c的参数来执行一个bash子进程命令。就像这个安全漏洞的测试脚本一样：

```ruby
$ env VAR='() { :;}; echo Bash is vulnerable!' bash -c "echo Bash Test"
```

其中，\(\) { :;} 中的冒号就相当于/bin/true，返回true并退出。而bash -c其实就是在spawn一个bash的echo的子进程，用于触发函数体外的echo命令。所以，更为友好一点的测试脚本应该是：

```ruby
env VAR='() { :;}; echo Bash is vulnerable!' bash -c "echo 如果你看到了vulnerable字样说明你的bash有安全问题"
```

OK，你应该明白这个漏洞是怎么一回事了吧。

  


### bash漏洞的影响有多大

在网上看到好多人说这个漏洞不大，还说这个事只有那些陈旧的执行CGI脚本的网站才会有，现在已经没有网站用CGI了。我靠，这真是无知者无畏啊。

我举个例子，如果你的网站中有调用操作系统的shell命令，比如你用PHP执行个exec之类的东西。这样的需求是有的，特别是对于一些需要和操作系统交互的重要的后台用于系统管理的程序。于是就会开一个bash的进程来执行。

我们还知道，现在的HTTP服务器基本上都是以子进程式的，所以，其中必然会存在export 一些环境变量的事，而有的环境变量的值是从用户端来的，比如：HTTP\_USER\_AGENT这样的环境变量，只由浏览器发出的。其实这个变量你想写成什么就写成什么。

于是，我可以把这个HTTP\_USER\_AGENT的环境变量设置成上述的测试脚本，只不过，我会把echo Bash is vulnerable!这个东西换成别的更为凶残的命令。呵呵。

关于这个漏洞会影响哪些已有的系统，你可以自己Google，几乎所有的报告这个漏洞的文章都说了（比如：[这篇](https://securityblog.redhat.com/2014/09/24/bash-specially-crafted-environment-variables-code-injection-attack/)，[这篇](https://www.digitalocean.com/community/tutorials/how-to-protect-your-server-against-the-shellshock-bash-vulnerability)），我这里就不复述了。

注：如果你要看看你的网站有没有这样的问题，你可以用这个在线工具测试一下：[‘ShellShock’ Bash Vulnerability CVE-2014-6271 Test Tool](http://shellshock.brandonpotter.com/)。

现在，你知道这事可能会很大了吧。还不赶快去打补丁。（注，yum update bash 把bash版本升级到 4.1.2-15.el6\_5.2 ，）



### 关于 AfterShock – CVE-2014-7169 测试脚本的解释

很多同学没有看懂下面这个测试脚本是什么意思，我这里解释一下。

```ruby
env X='() { (a)=>\' sh -c "echo date"; cat echo
```

* X='\(\) { \(a\)=
* 其中的 \(a\)=这个东西目的就是为了让bash的解释器出错（语法错误）。
* 语法出错后，在缓冲区中就会只剩下了 “&gt;\”这两个字符。
* 于是，这个神奇的bash会把后面的命令echo date换个行放到这个缓冲区中，然后执行。

相当于在shell 下执行了下面这个命令：

```ruby
$ >\
echo date
```

如果你了解bash，你会知道 \ 是用于命令行上换行的，于是相当于执行了：

```ruby
$ >echo date
```

这不就是一个重定向么？上述的命令相当于：

```ruby
$ date > echo
```

于是，你的当前目录下会出现一个echo的文件，这个文件的内容就是date命令的输出。

**能发现这个种玩法的人真是个变态，完全是为bash的源代码量身定制的一个攻击**。

