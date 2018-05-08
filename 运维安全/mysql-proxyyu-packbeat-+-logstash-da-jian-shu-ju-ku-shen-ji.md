# 一、介绍

packbeat：

```
 packbeat是一个开源的实时网络抓包与分析框架，内置了很多常见的协议捕获及解析，如HTTP、MySQL、Redis等。在实际使用中，通常和Elasticsearch以及kibana联合使用，用于数据搜索和分析以及数据展示。
```

mysql-proxy：

```
 mysql-proxy是mysql官方提供的mysql中间件服务，上游可接入若干个mysql-client，后端可连接若干个mysql-server。它使用mysql协议，任何使用mysql-client的上游无需修改任何代码，即可迁移至mysql-proxy上。
```

工作示例：![](/assets/mysql-proxy.png)

我们这里是把packbeat和mysql-proxy都安装在同一台服务器上，作为审计内部员工访问数据库的一种方法，性能应该不存在问题，毕竟并发不高。

# 二、mysql-proxy安装





