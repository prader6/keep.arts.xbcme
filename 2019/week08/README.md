## ARTS(week08)

### 算法题(Algorithm)

[移动零](https://github.com/geekwho11/learn.leetcode.xbcme/tree/master/php/src/283.move-zeroes)

### 阅读点评(Review)

#### [Consistent Nonlocking Reads](https://dev.mysql.com/doc/refman/8.0/en/innodb-consistent-read.html)

#### 阅读笔记

一致非锁定读取

一致性读意味着innodb使用多版本控制向查询提供数据库在某个时间点的快照。查询可以查看该时间点之前提交事务所做的更改，看不到以后或者未提交事务所做的更改。有一个例外，查询可以查看到同一个事务早期语句所做的更改。这个例外导致以下异常：如果更新表中的某些行，SELECT将查看更新行的最新版本，但它也可能会看到任何行的旧版本。如果其他会话同时更新同一个表，这个异常意味着你可能会看到该表处于从未存在于数据库中的状态。

如果事务隔离级别是REPEATABLE READ（默认级别），则同一事务中的所有一致性读取将读取该事务中第一次读取所建立的快照。您可以通过提交当前事务，然后提交新的查询，获取更新的快照。

使用READ COMMITTED隔离级别，事务中的每个一致读取都会设置并读取其自己的新快照。

一致性读取是InnoDB在READ COMMITTED和REPEATABLE READ隔离级别中处理SELECT语句的默认模式。一致读取不会对其访问的表设置任何锁定，因此其他会话可以在对表执行一致读取的同时自由修改这些表。

假设您正在以默认的REPEATABLE READ隔离级别运行。当您发出一致读取（即普通的SELECT语句）时，InnoDB会为您的事务提供一个时间点，您的查询将根据该时间点查看数据库。如果另一个事务删除了一行并在分配了时间点后提交，则不会将该行视为已删除。插入和更新的处理方式类似。

备注

数据库状态的快照适用于事务中的SELECT语句，不一定适用于DML(Data manipulation language 数据操作语言)语句。如果你插入或修改某些行然后提交该事务，则从另一个并发REPEATABLE READ事务发送的DELETE或UPDATE语句可能会影响那些刚刚提交的行，即使会话无法查询它们。如果事务确实更新或删除了由其他事务提交的行，则这些更改将对当前事务可见。例如，您可能会遇到如下情况：

```
SELECT COUNT(c1) FROM t1 WHERE c1 = 'xyz';
-- Returns 0: no rows match.
DELETE FROM t1 WHERE c1 = 'xyz';
-- Deletes several rows recently committed by other transaction.

SELECT COUNT(c2) FROM t1 WHERE c2 = 'abc';
-- Returns 0: no rows match.
UPDATE t1 SET c2 = 'cba' WHERE c2 = 'abc';
-- Affects 10 rows: another txn just committed 10 rows with 'abc' values.
SELECT COUNT(c2) FROM t1 WHERE c2 = 'cba';
-- Returns 10: this txn can now see the rows it just updated.
```

您可以通过提交您的事务然后再进行另一次SELECT或[`START TRANSACTION WITH CONSISTENT SNAPSHOT`](https://dev.mysql.com/doc/refman/8.0/en/commit.html)来获取最新时间的快照。

这称为多版本并发控制。

在下面的示例中，会话A仅在B已提交插入且A已提交时才看到由B插入的行，以便获取会话B提交的最新快照。

如果你想要查看数据库的最新状态，可以使用 READ Commited隔离级别或者锁定读取。

```
SELECT * FROM t FOR SHARE;
```

在READ COMMITTED隔离级别，事务中的每个一致读取都会设置并读取其自己的新快照。使用FOR SHARE时，会发生锁定读取：SELECT阻塞，直到包含事务结束的最新行。

一致读取对某些DDL(Data definition language 数据定义语言)语句不起作用：

1. 一致读取不适用于DROP TABLE，因为MySQL无法使用已删除的表并且InnoDB会破坏该表。
2. 一致性读取不适用于ALTER TABLE，因为该语句生成原始表的临时副本，并在构建完临时副本后删除原始表。在事务中重新发送一致读取时，新表中的行不可见，因为在获取事务的快照时这些行不存在。在这种情况下，事务返回错误：ER_TABLE_DEF_CHANGED，“表定义已更改，请重试事务”。

读取类型因INSERT INTO ... SELECT，UPDATE ...（SELECT）和CREATE TABLE ... SELECT等子句中的选择而异，不指定FOR UPDATE或FOR SHARE：

1. 默认情况下，InnoDB使用更高级的锁定，而SELECT部分的作用类似于READ COMMITTED，即使在同一事务中，每个一致的读取也会设置和读取自己的新快照。
2. 要在这种情况下使用一致读取，请将事务的隔离级别设置为READ UNCOMMITTED，READ COMMITTED或REPEATABLE READ（即SERIALIZABLE以外的任何其他内容）。在这种情况下，不会对从所选表中读取的行设置锁定。

### 技术技巧(Tip)

#### SSH 代理

#### 参数

```
-vvv 输出详细的调试日志，最大为3，示例 -v -vv -vvv。
-C 请求压缩所有数据（包括stdin，stdout，stderr和X11转发的数据，TCP和UNIX域连接）。压缩算法是gzip。
-N 不要执行远程命令。 这对于转发端口很有用。
-g 允许远程主机连接到本地转发端口。 如果用于多路复用连接，然后必须在主进程上指定此选项
-L 本地端口号:远程主机:远程主机端口号 -p (连接远程主机的端口) 连接用户名@连接的主机
-f 请求ssh在命令执行之前转到后台。
-D 绑定的本地的端口号
```

通过用户名ssh，端口2211连接到remote.xbc.me，转发服务器端口3306到本地3306，这样，本地的3306的端口和服务器的3306是同一个端口。

#### 调试命令

```
ssh -vvv -CNg -L 3306:remote.xbc.me:3306 -p 2211 ssh@remote.xbc.me
```

#### 后台运行

```
ssh -CfNg -L 3306:remote.xbc.me:3306 -p 2211 ssh@remote.xbc.me
后台守护进程
nohup ssh -CfNg -L 3306:remote.xbc.me:3306 -p 2211 ssh@remote.xbc.me >> ~/ssh.log &
```

#### AutoSSH

autossh的解决了下面两个问题：

1. 处理ssh进程被关闭的问题，也可以尝试用ssh的-N参数解决
2. 由于网络故障/波动SSH链接中断时，无法自动重连。

#### 参数

```
-M 2011 选项指定中继服务器上的监视端口，用于交换监视 SSH 会话的测试数据，需要保证该端口在服务器上未被占用。
-f 指定 autossh 在后台运行，并不会传给 ssh。和 ssh 的 -f 不一样，autossh 指定 -f 时将无法输入密码。
autossh [-V] [-M port[:echo_port]] [-f] [SSH_OPTIONS] 其他参数会传递给ssh
```

autossh实现自动重连。

```
autossh -M 2011 -CfNg -L 3306:db.xbc.me:3306 -p 2211 ssh@remote.xbc.me >> ~/autossh.log
后台守护进程
nohup autossh -M 2011 -CfNg -L 3306:db.xbc.me:3306 -p 2211 ssh@remote.xbc.me >> ~/autossh.log &

连接到主机，并绑定本地的端口8080，转发相关网络请求
autossh -M 2021 -N -v ssh@remote.xbc.me -D 0.0.0.0:8080
```

#### 参考链接

1. [SSH端口转发](https://www.cnblogs.com/zangfans/p/8848279.html)
2. [使用 autossh 建立反向 SSH 隧道管理个人计算机](https://blog.windrunner.me/sa/reverse-ssh.html)

### 分享(Share)

#### [JSON Web Token 入门教程](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html)

#### 阅读笔记

##### 应用场景

一种解决方案是 session 数据持久化，写入数据库或别的持久层。各种服务收到请求后，都向持久层请求数据。这种方案的优点是架构清晰，缺点是工程量比较大。另外，持久层万一挂了，就会单点失败。

另一种方案是服务器索性不保存 session 数据了，所有数据都保存在客户端，每次请求都发回服务器。JWT 就是这种方案的一个代表。

##### 数据结构

```
Header.Payload.Signature
```

Header

```
{
  "alg": "HS256",
  "typ": "JWT"
}
上面代码中，alg属性表示签名的算法（algorithm），默认是 HMAC SHA256（写成 HS256）；typ属性表示这个令牌（token）的类型（type），JWT 令牌统一写为JWT。
```

Payload

```
可用字段
iss (issuer)：签发人
exp (expiration time)：过期时间
sub (subject)：主题
aud (audience)：受众
nbf (Not Before)：生效时间
iat (Issued At)：签发时间
jti (JWT ID)：编号
例子
{
  "sub": "1234567890",
  "name": "John Doe",
  "admin": true
}
```

注意，JWT 默认是不加密的，任何人都可以读到，所以不要把秘密信息放在这个部分。这个 JSON 对象也要使用 Base64URL 算法转成字符串。

Signature 部分是对前两部分的签名，防止数据篡改。

```
HMACSHA256(
  base64UrlEncode(header) + "." +
  base64UrlEncode(payload),
  secret)
```

JWT 的使用方式

1. cookie 无法跨域

2. localStorage

3. ```
   Authorization: Bearer <token>
   ```

4. 跨域的时候，JWT 就放在 POST 请求的数据体里面。

JWT 的几个特点

1. JWT 默认是不加密，但也是可以加密的。生成原始 Token 以后，可以用密钥再加密一次。
2. JWT 不加密的情况下，不能将秘密数据写入 JWT。
3. JWT 不仅可以用于认证，也可以用于交换信息。有效使用 JWT，可以降低服务器查询数据库的次数。
4. JWT 的最大缺点是，由于服务器不保存 session 状态，因此无法在使用过程中废止某个 token，或者更改 token 的权限。也就是说，一旦 JWT 签发了，在到期之前就会始终有效，除非服务器部署额外的逻辑。
5. JWT 本身包含了认证信息，一旦泄露，任何人都可以获得该令牌的所有权限。为了减少盗用，JWT 的有效期应该设置得比较短。对于一些比较重要的权限，使用时应该再次对用户进行认证。
6. 为了减少盗用，JWT 不应该使用 HTTP 协议明码传输，要使用 HTTPS 协议传输。

#### 参考链接

1. [JSON Web Token 入门教程](https://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html)

#### 思考总结

1. 由Header、Payload、Signature的三部分构成。
2. 自定义签名，服务器可以做成无状态，方便扩展。
3. 签名算法 HMAC SHA256 字符串需要进行Base64URL算法处理。
4. 涉及认证的盗用，特别是关于用户敏感信息，需要做二次验证，例如GA两步认证、手机短信、App客户端确认等方式。