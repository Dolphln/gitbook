# 事件来源

8 月 21 号，Tavis Ormandy 通过公开邮件列表，再次指出 GhostScript 的安全沙箱可以被绕过，通过构造恶意的图片内容，将可以造成命令执行、文件读取、文件删除等漏洞：

http://seclists.org/oss-sec/2018/q3/142

  


GhostScript 被许多图片处理库所使用，如 ImageMagick、Python PIL 等，默认情况下这些库会根据图片的内容将其分发给不同的处理方法，其中就包括 GhostScript。

  


ImageMagick 是一款广泛使用的图像处理软件，有相当多的网站使用它来进行图像处理。它被许多编程语言所支持，包括 Perl，C++，PHP，Python 和 Ruby 等，并被部署在数以百万计的网站，博客，社交媒体平台和流行的内容管理系统（CMS），例如 WordPress 和 Drupal。

  


Python PIL 是 Python 语言中处理图片的第三方模块。

  


