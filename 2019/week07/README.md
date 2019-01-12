## ARTS(week07)

### 算法题(Algorithm)

[加一](https://github.com/geekwho11/learn.leetcode.xbcme/tree/master/php/src/66.plus-one)

### 阅读点评(Review)

#### [InnoDB Locking](https://dev.mysql.com/doc/refman/8.0/en/innodb-locking.html)

#### 阅读笔记

这一节描述了innodb的锁类型

1. 共享锁和排它锁
2. 意向锁
3. 行锁
4. 间隙锁
5. 下一键锁
6. 插入意向锁
7. 自增长锁
8. 空间索引的断言锁

#### 共享和排它锁

InnoDB实现标准的行级锁定，其中有两种类型的锁，共享S和独占(排它)锁。

1. 共享锁(S)允许持有锁的事务读取行。
2. 独占锁(X)允许持有锁的事务更新或删除行。

如果事务t1在行r上保持共享锁，则另外一个不同的事务t2对行r上的锁处理如下：

1. 对t2的S锁请求可以立即授予。结果，t1和t2都在行r上保持共享锁。
2. t2的独占锁请求无法立即授予。

如果事务t1在行r上保持独占锁，则另外一个不同的事务t2无法获得任一类型的锁定请求。相反，事务t2必须等待事务t1处理完释放其对行r的锁定。

#### 意向锁

InnoDB支持多个粒度锁定，允许行锁和表锁共存。例如，lock tables write等语句在指定的表上采用独占锁。为了实现多个粒度级别的锁定，InnoDB使用意向锁。意向锁是表级别的锁定，它标明了事务在稍后对表中的行所需要的锁类型。意向锁有2种类型:

1. 意向共享锁(IS)表示事务打算在表的独立行上设置一个共享锁。
2. 意向独占锁(IX)表示事务打算在表的独立行上设置一个独占锁。

例如， select for share 设置 IS锁，select for update 设置IX锁。

意向锁的锁定协定下：

1. 在事务可以获取表中某行的共享锁之前，它必须先要在表上获取IS锁或者更高级别的锁。
2. 在事务可以获取表中某行的独占锁之前，它必须先要在表上获取独占锁。

表级别锁类型兼容性总结在以下的列表：

|      | `X`      | `IX`       | `S`        | `IS`       |
| ---- | -------- | ---------- | ---------- | ---------- |
| `X`  | Conflict | Conflict   | Conflict   | Conflict   |
| `IX` | Conflict | Compatible | Conflict   | Compatible |
| `S`  | Conflict | Conflict   | Compatible | Compatible |
| `IS` | Conflict | Compatible | Compatible | Compatible |

如果请求事务的锁与现有锁可以兼容，那么就可以给予锁，如果有冲突的话，就无法获得锁。事务必须等待与其有冲突的现有锁释放后，才能获取锁。如果锁定请求与现有锁有冲突，因为可能会发生死锁，导致错误，所以无法获取到锁。意向锁几乎不会阻塞，除了全表请求(例如，lock table write)。意向锁的主要目的是显示某人正在锁定行，或者正要锁定表中的某行。意向锁的事务数据与show engine innodb status 和 InnoDB监视器类似：

```
TABLE LOCK table `test`.`t` trx id 10080 lock mode IX
```

#### 行锁

行锁是对索引记录的锁定。例如，select c1 from t where c1=10 for update。阻止任何其他事务插入，更新或者删除t.c1的值为10的行。行锁始终锁定索引记录，即使表没有定义索引。对于这种情况，InnoDB会创建一个隐藏的聚集索引并使用这个索引记录进行锁定。

#### 间隙锁/间隔锁

间隔锁是锁定索引记录之间的间隔，或锁定在第一条记录之前或者最后一个索引记录之后的间隔上。例如，SELECT c1 FROM t WHERE c1 BETWEEN 10 and 20 FOR UPDATE; 会阻止其他事务将值15插入到t.c1中，无论列中是否存在任何值，因为该范围内的之间的间隔都被锁定。

间隔可能跨越单个索引值，多个索引值，甚至可能为空。

间隔锁是性能和并发的一种平衡，用于某些事务隔离级别而不是其他级别。

使用唯一索引锁定以搜索唯一行的语句，不需要间隔锁定。(这不包括搜索条件仅包含多列唯一索引的某些列的情况;在这种情况下，确实会发生间隔锁定。)。例如，如果id列具有唯一索引，则以下语句仅锁定行的值为100的索引记录。其他会话是否在前一个间隔插入行是无关紧要的。

```
SELECT * FROM child WHERE id = 100;
```

如果id没有索引或者不是唯一索引，则该语句或锁定前一个间隔。

这里还值得注意的是，冲突锁可以通过不同的事务保持在间隔上。例如，事务A可以在间隔保持共享间隔锁定，而事务B在同一间隔上保持独占间隔锁定。允许冲突间隔锁定的原因是，如果从索引中清除记录，则必须合并由不同事务保留在记录上的间隔锁定。

InnoDB的间隔锁是纯碎的防止。这意味着他们唯一目的是防止其他事务插入间隔。间隔所可以共存。一个事务占用的间隔锁定不会阻止另外一个事务在同一个间隔上进行的间隔锁定。共享和独占间隔锁之间没有区别。它们彼此不冲突，它们执行相同的功能。

可以明确的经用间隔锁定。如果将事务隔离级别更改为read commited，则会发生这种情况。在这些情况下，对于搜索和索引扫描禁用间隔锁定。，并且仅用于外键约束检查和重复键检查。

使用read commited的隔离级别还有其他影响。MySQL评估Where条件后，将释放非匹配行的行锁。对于update语句，InnoDB执行 半一致读取，以便将最新提交的版本返回给MySQL，以便MySQL可以确定该行是否与update的where条件匹配。

#### 下一键锁

下一键锁定是索引记录上的行锁和索引记录之前的间隔上的间隔锁组合。

InnoDB是这样的方式执行对行级锁定的：

当它搜索或者扫描表索引时，它会在遇到的索引记录上设置共享锁或独占锁。因此，行级锁实际上索引记录锁。索引记录的下一键锁定也会影响该索引记录之前的间隔，也就是说，下一键锁定是索引记录锁定假话是哪个索引记录之前间隔的间隔锁定。如果一个回话在索引中的记录R上具有共享锁或独占锁，则另外一个回话不能在索引顺序的R之前的间隔中插入新的索引记录。

假设索引包含值10,11,13,20.此索引的下一键索引覆盖以下区间，大约有5个区间。其中圆括号表示大于且不包含端点，方括号表示小于等于且包含端点。

```
(negative infinity, 10] <=10
(10, 11] >10 <= 11
(11, 13] >11 <= 13
(13, 20] >13 <= 20
(20, positive infinity) >20
```

对于最后一件间隔，下一件锁定将间隔锁定在索引中最大值智商，而假设的记录大于索引中实际的任何值。假设不是真正的索引记录，因此，实际上，此下一键锁定仅锁定最大索引值之后的间隔。

默认情况下，InnoDB在REPEATABLE READ事务隔离级别运行。在这种情况下，InnoDB使用下一件所进行搜索和索引扫描，从而防止幻影行。

#### 插入意向锁

插入意向锁是在行插入之前由INSERT操作设置的一种间隔锁定。该锁定表示以这样的方式插入：如果插入到相同索引间隔中的多个事务不插入间隔内的相同位置，则不需要等待彼此。假设存在值为4和7的索引记录，分别尝试插入值5和6两个独立的事务，在插入行上获得独占锁前，预先获取插入意向锁。因为这些行是没有冲突的，所以不会相互阻塞。

以下示例演示了在获取插入记录的独占锁之前采用插入意向锁的事务。该事务涉及2个客户端A和B。

客户端A创建一个包含两个索引记录（90和102）的表，然后启动一个事务，该事务对ID大于100的索引记录放置独占锁。独占锁包括记录102之前的间隙锁：

```
mysql> CREATE TABLE child (id int(11) NOT NULL, PRIMARY KEY(id)) ENGINE=InnoDB;
mysql> INSERT INTO child (id) values (90),(102);

mysql> START TRANSACTION;
mysql> SELECT * FROM child WHERE id > 100 FOR UPDATE;
+-----+
| id  |
+-----+
| 102 |
+-----+
```

客户端B开始一个事务以将记录插入间隙。 该事务在等待获取独占锁时采用插入意向锁。

```
mysql> START TRANSACTION;
mysql> INSERT INTO child (id) VALUES (101);
```

可以看到该事务一直处于等待中

```
RECORD LOCKS space id 31 page no 3 n bits 72 index `PRIMARY` of table `test`.`child`
trx id 8731 lock_mode X locks gap before rec insert intention waiting
Record lock, heap no 3 PHYSICAL RECORD: n_fields 3; compact format; info bits 0
```

#### 自增长锁

自增长锁是由插入到有AUTO_INCREMENT列表中的事务，所采用的特殊表级锁。在最简单的情况下，如果一个事务正在向表中插入值，则任何其他事务必须等待自己对该表的插入，以便第一个事务插入的行接收到连续的主键值。

innodb_autoinc_lock_mode配置选项控制用于自增长值的锁定算法。 它允许你对可预测的自增长值与插入操作并发最大化做权衡。

#### 空间索引的断言锁

InnoDB支持列包含空间列的空间索引。

为了处理操作中涉及空间索引的锁定，下一键锁在不可重复读或者串行化的事务隔离级别下工作得不是很好。在多维数据中没有绝对的排序概念，因此不清楚哪一个是"下一个"键。

为了在空间索引上支持隔离级别，InnoDB采用了断言锁。空间索引包含了最小边界矩形MBR的值，因此InnoDB通过在用于查询的MBR值上设置一个断言锁来强制对索引进行一致性读取。其他的事务无法插入或修改与查询条件匹配的行。

#### 思考总结

1. 如果解决了锁的问题，是否就可以提升整体高并发性能？
2. 还有其他的办法可以实现吗？

### 技术技巧(Tip)

修改提交者的邮箱。

```
# 重置全部的作者
git filter-branch -f --env-filter "
    GIT_AUTHOR_NAME='GeekWho'
    GIT_AUTHOR_EMAIL='geekwho@xbc.me'
    GIT_COMMITTER_NAME='GeekWho'
    GIT_COMMITTER_EMAIL='geekwho@xbc.me'
  " HEAD
# 重置部分作者
1. git rebase -i HEAD~3 找到你需要修改的提交
2. 将前缀的pick改为 e
3. git commit --amend --author="GeekWho <geekwho@xbc.me>"
```

#### 参考链接

1. [How to change the commit author for one specific commit?](https://stackoverflow.com/questions/3042437/how-to-change-the-commit-author-for-one-specific-commit)

### 分享(Share)

#### [Netflix有哪些来之不易的系统高可用经验](https://mp.weixin.qq.com/s/rkuwqPELKNaTjHZInY5AXw)

#### 阅读笔记

优先考虑区域部署而不是全球部署

使用红黑部署策略进行生产部署

使用部署窗口

不要在非工作时段或周末自动触发部署

启用 Chaos Monkey

在将代码推送到生产环境之前使用各种测试和金丝雀分析来验证代码

必要的人工干预

部署时尽可能只用已经测试过的东西

定期检查联系人设置

知道如何快速回滚部署

如果实例运行不正常，将部署视为失败

在进行自动部署时，需要通知团队有关部署的情况

自动化非典型部署，而不是进行一次性手动部署

使用先决条件验证预期状态

#### 思考总结

1. 这里的每一条都是实践出来的经验吧。
2. 其实就是把发布更新的标准，再提高一步，这和我之前的原则，几乎是一致的，没有经过测试的代码，不要运行在生产环境，除非你确认不会对生产环境的用户产生影响。

#### 参考链接
1. [Tips for High Availability](https://medium.com/@NetflixTechBlog/tips-for-high-availability-be0472f2599c)