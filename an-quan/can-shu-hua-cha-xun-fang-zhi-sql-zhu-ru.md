# 参数化查询防止sql注入

预编译之所以能防止sql注入（我在这里只说sql注入这方面），还要从sql的执行方式有关。

一般来说，sql 是怎么执行的呢？简单说，一个sql 是经过解析器编译并执行，注意这里是一个并字。

举一个栗子,校验有没有这个用户的场景sql

select count\(1\) from students where name='张三'

上边的数据库执行时，是直接将这句话连带 name='张三'，一起给编译了，然后执行。假设我的sql 注入语句是

select count\(1\) from students where name='张三' or '1=1'

即name 参数为张三‘ or '1=1 ，这个参数也会被编译器一同编译。

**而使用预编译，数据库是怎么处理的呢？**

这个步骤是在con.prepareStatement（“”）语句执行的时候，服务器就这个sql发送给了数据库，然后数据库将该sql编译后放入到缓存池中。等到服务器执行execute的时候，传给数据库的 张三' or '1=1 编译器并不进行编译，而是这找到原来的sql模板，传参，执行。所以， 张三‘ or '1=1 只会被数据库当做参数来处理。



**简单总结，参数化能防注入的原因在于，语句是语句，参数是参数，参数的值并不是语句的一部分，数据库只按语句的语义跑，至于跑的时候是带一个普通背包还是一个怪物，不会影响行进路线，无非跑的快点与慢点的区别。** 



**示例：**

使用预编译处理之后的

SELECT \* FROM test order by 'name \ ' or \ '1=\ '1'

SELECT \* FROM test order by 'id and 1=1'

![](/assets/sql-1.png)



参考资料：

[https://blog.csdn.net/yan465942872/article/details/6753957](https://blog.csdn.net/yan465942872/article/details/6753957)      占位符，SQL注入？（这一篇从mysql的jar包找到了源码说明）

[https://www.zhihu.com/question/52869762](https://www.zhihu.com/question/52869762)      为什么参数化SQL查询可以防止SQL注入?

