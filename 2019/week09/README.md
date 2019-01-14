## ARTS(week09)

### 算法题(Algorithm)

[两数之和](https://github.com/geekwho11/learn.leetcode.xbcme/tree/master/php/src/1.two-sum)

### 阅读点评(Review)

#### [Locking Reads](https://dev.mysql.com/doc/refman/8.0/en/innodb-locking-reads.html)

#### 阅读笔记

如果查询数据然后在同一事务中插入或更新相关数据，则常规SELECT语句不会提供足够的保护。 其他事务可以更新或删除您刚查询的相同行。 InnoDB支持两种类型的锁定读取，提供额外的安全性：

1. [`SELECT ... FOR SHARE`](https://dev.mysql.com/doc/refman/8.0/en/select.html)

在读取的任何行上设置共享模式锁定。 其他会话可以读取行，但在事务提交之前无法修改它们。 如果这些行中的任何行已被另一个尚未提交的事务更改，则查询将等待该事务结束，然后使用最新值。

备注

SELECT ... FOR SHARE是SELECT ... LOCK IN SHARE MODE的替代品，但LOCK IN SHARE MODE仍可用于向后兼容。 这些陈述是等同的。 但是，FOR SHARE支持OF table_name，NOWAIT和SKIP LOCKED选项。

1. [`SELECT ... FOR UPDATE`](https://dev.mysql.com/doc/refman/8.0/en/select.html)

对于搜索遇到的索引记录，锁定行和任何关联的索引条目，就像为这些行发出UPDATE语句一样。 阻止其他事务更新这些行，从执行SELECT ... FOR SHARE，或从某些事务隔离级别读取数据。 一致性读取将忽略在读取视图中存在的记录上设置的任何锁定。 （旧版本的记录无法锁定;它们是通过在记录的内存副本上应用撤消日志来重建的。）

在处理树形结构或图形结构数据时，这些子句主要用于单个表中或跨多个表分割。 您可以将边缘或树枝从一个地方遍历到另一个地方，同时保留返回的权限并更改任何这些“指针”值。

在提交或回滚事务时，将释放由FOR SHARE和FOR UPDATE查询设置的所有锁。

备注

仅当禁用自动提交时（通过使用START TRANSACTION开始事务或将自动提交设置为0），才能锁定读取。

除非在子查询中指定了锁定读取子句，否则外部语句中的锁定读取子句不会锁定嵌套子查询中的表行。 例如，以下语句不会锁定表t2中的行。

```
SELECT * FROM t1 WHERE c1 = (SELECT c1 FROM t2) FOR UPDATE;
```

要锁定表t2中的行，请在子查询中添加一个锁定读取子句：

```
SELECT * FROM t1 WHERE c1 = (SELECT c1 FROM t2 FOR UPDATE) FOR UPDATE;
```

#### 锁定读取示例

假设您要在表子项中插入新行，并确保子行在表父项中具有父行。 您的应用程序代码可确保整个操作序列中的引用完整性。

首先，使用一致性读取来查询表PARENT并验证父行是否存在。 你能安全地将子行插入表CHILD吗？ 不，因为其他一些会话可以删除SELECT和INSERT之间的父行，而不会意识到它。

要避免此潜在问题，请使用FOR SHARE执行SELECT：

```
SELECT * FROM parent WHERE NAME = 'Jones' FOR SHARE;
```

在FOR SHARE查询返回父“Jones”之后，您可以安全地将子记录添加到CHILD表并提交事务。尝试获取PARENT表中适用行的独占锁的任何事务都会等到完成后，即直到所有表中的数据都处于一致状态。

再举一个例子，考虑表CHILD_CODES中的整数计数器字段，用于为添加到表CHILD的每个子节点分配唯一的标识符。不要使用一致读取或共享模式读取来读取计数器的当前值，因为数据库的两个用户可以看到计数器的相同值，并且如果两个事务尝试添加行，则会发生重复键错误与CHILD表相同的标识符。

这里，FOR SHARE不是一个好的解决方案，因为如果两个用户同时读取计数器，则当它们尝试更新计数器时，其中至少有一个会陷入死锁。

要实现读取和递增计数器，首先使用FOR UPDATE执行计数器的锁定读取，然后递增计数器。例如：

```
SELECT counter_field FROM child_codes FOR UPDATE;
UPDATE child_codes SET counter_field = counter_field + 1;
```

SELECT ... FOR UPDATE读取最新的可用数据，在它读取的每一行上设置独占锁。 因此，它设置搜索的SQL UPDATE将在行上设置的相同锁。

前面的描述仅仅是SELECT ... FOR UPDATE如何工作的一个例子。 在MySQL中，生成唯一标识符的具体任务实际上只需对表进行一次访问即可完成：

```
UPDATE child_codes SET counter_field = LAST_INSERT_ID(counter_field + 1);
SELECT LAST_INSERT_ID();
```

SELECT语句仅检索标识符信息（特定于当前连接）。 它不访问任何表。

##### 使用NOWAIT和SKIP LOCKED锁定读取并发

如果行被事务锁定，则请求相同锁定行的SELECT ... FOR UPDATE或SELECT ... FOR SHARE事务必须等到阻塞事务释放行锁。 此行为可防止事务更新或删除由其他事务查询以进行更新的行。 但是，如果您希望在请求的行被锁定时立即返回查询，或者从结果集中排除锁定的行是可接受的，则无需等待释放行锁定。

为了避免等待其他事务释放行锁，NOWAIT和SKIP LOCKED选项可以与SELECT ... FOR UPDATE或SELECT ... FOR SHARE锁定读取语句一起使用。

1. NOWAIT

   使用NOWAIT的锁定读取永远不会等待获取行锁定。 查询立即执行，如果请求的行被锁定，则失败并显示错误。

2. SKIP LOCKED

使用SKIP LOCKED的锁定读取永远不会等待获取行锁定。 查询立即执行，从结果集中删除锁定的行。

备注

跳过锁定行的查询会返回数据的不一致视图。 因此，SKIP LOCKED不适用于一般交易工作。 但是，当多个会话访问同一个类似队列的表时，它可用于避免锁争用。

NOWAIT和SKIP LOCKED仅适用于行级锁。

使用NOWAIT或SKIP LOCKED的语句对于基于语句的复制是不安全的。

以下示例演示NOWAIT和SKIP LOCKED。 会话1启动一个事务，该事务对单个记录执行行锁定。 会话2尝试使用NOWAIT选项对同一记录进行锁定读取。 由于请求的行被会话1锁定，因此锁定读取会立即返回错误。 在会话3中，使用SKIP LOCKED进行的锁定读取将返回请求的行，但会话1锁定的行除外。

```
# Session 1:

mysql> CREATE TABLE t (i INT, PRIMARY KEY (i)) ENGINE = InnoDB;

mysql> INSERT INTO t (i) VALUES(1),(2),(3);

mysql> START TRANSACTION;

mysql> SELECT * FROM t WHERE i = 2 FOR UPDATE;
+---+
| i |
+---+
| 2 |
+---+

# Session 2:

mysql> START TRANSACTION;

mysql> SELECT * FROM t WHERE i = 2 FOR UPDATE NOWAIT;
ERROR 3572 (HY000): Do not wait for lock.

# Session 3:

mysql> START TRANSACTION;

mysql> SELECT * FROM t FOR UPDATE SKIP LOCKED;
+---+
| i |
+---+
| 1 |
| 3 |
+---+
```

###  技术技巧(Tip)

#### PHP超实用系列·自动捕获Fatal Error

| 函数                         | 应用范围                                     | 备注   |
| -------------------------- | ---------------------------------------- | ---- |
| set_exception_handler      | 可以捕获程序中抛出的异常。                            |      |
| set_error_handler          | E_USER_ERROR、 E_USER_WARNING、 E_USER_NOTICE |      |
| register_shutdown_function | 注册一个 `callback` ，它会在脚本执行完成或者 [exit()](http://php.net/manual/zh/function.exit.php) 后被调用。 |      |
| error_get_last             | 获取最后发生的错误                                |      |

set_error_handler函数无法捕获以下错误：

```
以下级别的错误不能由用户定义的函数来处理： E_ERROR、 E_PARSE、 E_CORE_ERROR、 E_CORE_WARNING、E_COMPILE_ERROR、 E_COMPILE_WARNING，和在 调用 set_error_handler() 函数所在文件中产生的大多数E_STRICT。
```

register_shutdown_function执行情况

1. 脚本正常退出
2. 脚本运行时出现退出，parse time 解析错误不会触发。
3. 用户调用exit退出

#### 参考连接

1. [PHP超实用系列·自动捕获Fatal Error](https://segmentfault.com/a/1190000009315317)
2. [set_error_handler](http://php.net/manual/zh/function.set-error-handler.php)
3. [register_shutdown_function](http://php.net/manual/zh/function.register-shutdown-function.php)

### 分享(Share)

#### [PHP 进阶之路 - 深入理解 FastCGI 协议以及在 PHP 中的实现](https://segmentfault.com/a/1190000009863108)

#### 阅读笔记

文章非常清晰的描述了CGI和FastCGI协议的差异。

##### CGI

1. 通用网关接口
2. 优点是该协议运行在HTTP协议之上，屏蔽执行CGI程序的差异，只需要按照CGI协议的标准输入和标准输出来处理客户端的请求即可。
3. 缺点是每次都需要fork一个子进程处理CGI程序

#####  FastCGI

1. 改进后的CGI协议
2. 优点是取缔传统的 fork-and-execute 方式，减少每次启动的巨大开销（后面以 PHP 为例说明），以常驻的方式来处理请求。
3. 简单的来说，带有进程池+常驻后台的服务，提升处理请求的性能。

#####  PHP 中的 FastCGI 的实现

对PHP的标准库进行详细的解释。值得一看。

#### 参考链接

1. [The Common Gateway Interface (CGI) Version 1.1](https://datatracker.ietf.org/doc/rfc3875/?include_text=1)
2. [FastCGI规范](https://andylin02.iteye.com/blog/648412)