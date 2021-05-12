---
  title: 支持后台编写markDown的hexo
  tags:
---
  
**keyword：行锁，二阶段锁，死锁

mysql的行锁是由引擎层各个引擎实现的，并不是所有的引擎都支持行锁，例如MyIsam就不支持行锁，这就意味着MyIsam只能使用表锁，同一张表同一时间只能有一个操作，这也是InnoDB替代MyIsam的主要原因

若一个事务内有多条更新语句操作多条行，I**nnoDB采用每条更新语句加一个行锁，最终事务提交后再释放所有的锁，这被称作为二阶段锁。**

假设id为表t的主键，那么下图中只有事务A执行完后事务B才会执行

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/1f02d479-f39b-4a70-b9b1-282d05a5ed23/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/1f02d479-f39b-4a70-b9b1-282d05a5ed23/Untitled.png)

因此如果事务中锁住了多行，那么可以把最可能造成冲突的，最可能影响并发度的放在最后，比如：

1. 从顾客A账户余额中扣除电影票价；
2. 给影院B的账户余额增加这张电影票价；
3. 记录一条交易日志。

在这个例子中，步骤2的操作最容易造成冲突，例如影院B可能有多个人购买，因此可以将步骤2移动到步骤3

### 死锁

mysql死锁是事务A持有id = 1的行锁在等待id = 2的行锁，事务B持有id = 2的行锁在等待id = 1的行锁

![https://s3-us-west-2.amazonaws.com/secure.notion-static.com/37a7d2cb-2ae1-4869-be02-cef4c9037b74/Untitled.png](https://s3-us-west-2.amazonaws.com/secure.notion-static.com/37a7d2cb-2ae1-4869-be02-cef4c9037b74/Untitled.png)

当发生死锁时一般有两种策略：

1. 等待，直到超时，这个超时时间可以通过参数innodb_lock_wait_timeout来设置
2. 主动检查，回滚其中一个事务，将参数innodb_deadlock_detect设置为on

对于策略1，超时时间太长会导致数据库操作时间太长，超时时间太短时，如果不是死锁，而是简单等待锁释放，这种情况下会误操作，因此一般采用第二种策略

### 死锁检测

主动死锁检测是发生在事务发生时，每个加锁操作都要检测自身的加入会不会造成死锁，这是一个时间复杂度为O(n)的操作。假设有1000个并发线程要同时更新一行，那么死锁检测操作就是100万级别的，这是非常耗费CPU的

1. 如果能确定不会出现死锁，可以把死锁检测关闭
2. 控制并发度：比如在某一行只有10个线程更新，那么死锁检测的成本就很低。一种方法是在客户端做控制，这种方式如果客户端较多的话就gg了，因此这种需要在服务端做限制，如果有中间件就可以用中间件。没有的话就需要在mysql层面操作

另外还可以考虑将一行换成逻辑的多行，例如总额一行可以改为多行相加
