## ARTS(week05)

### 算法题(Algorithm)

[只出现一次的数字\](https://github.com/geekwho11/learn.leetcode.xbcme/blob/master/php/src/136.single-number)

### 阅读点评(Review)

#### [MHA Quick Start Guide](https://dzone.com/articles/mha-quick-start-guide?fromrel=true)

#### 阅读笔记

什么是MHA?Master High Availability Manager and tools for MySQL。

MHA是重要的服务管理一环。正确的设置后，可以检查复制运行状态，通过VIP迁移读写，执行故障转移，持续监视Nagios。它易于部署，并遵从我非常喜欢的KISS原则。

这篇文章是一个快速入门指南，你可以在自己的测试环境中尝试并使用它。前提是你已经了解如何安装软件，处理SSH keys和设置MySQL复制。

前提条件

1. 复制必须已经运行中。MHA管理和监控复制的运行，该工具无法进行部署。因此MySQL和复制需要已经在运行中。
2. 所有的主机都需要通过SSH keys进行连接。
3. 所有的节点必须可以连接MySQL服务器。
4. 所有的节点都必须有相同的用户名和密码。
5. 在多主的模式下，只允许设置一个节点可以写入。其他的节点需要配置为可读。
6. MySQL的版本配置在5.0以上
7. 备用服务器必须启用二进制日志，方便故障转移后进行二进制日志复制。复制的用户也需要存在。
8. 所有服务器上的二进制过滤变量都必须相同 (replicate-wild%, binlog-do-db…)。
9. 禁用自动清理中继日志，并且配置为定时任务定期删除。你可以使用MHA自带的脚本purge_relay_logs进行定期删除。

你可以测试手动执行故障转移，还需要在master server 做部分手动操作。

1. 停止masterha_manager

2. ```
   masterha_master_switch –master_state=alive –conf=/etc/app1.cnf
   ```

3. 执行上面的命令，代表你想要切换master，但是实际上master依然存活，因此不需要标识它挂掉或者从配置文件中移除。

一些有用的参数

1. ```
   –orig_master_is_new_slave: 把备机提升主服务

   –running_updates_limit: 如果当前主服务写入超时，或者从机也超时，默认会停止切换。默认超时为1s.这些都是从安全方面考虑。

   –interactive=0: 如果你想要跳过全部的确认请求。
   ```

### 技术技巧(Tip)

#### Linux常用技巧

su

```
su  username    //切换到指定的用户，不加载配置文件和环境变量。
su - username   //切换到指定的用户，加载配置文件和环境变量。
su - -c "touch /tmp/test.txt"  username //用指定用户的身份执行一条命令。
```

grep

```
# 找到配置指定域名的配置文件
grep "xbc.me" -rl *.conf
-l 参数只支持POSIX，只输出匹配的文件名
-r 循环去匹配当前的全部文件
```

#### 参考链接

1. [su命令切换用户](http://blog.51cto.com/11060853/2092490)

### 分享(Share)

#### [从零开始搭建创业公司后台技术栈](http://www.phppan.com/2018/04/svr-stack/)

#### 阅读笔记

##### 各系统组件选型

1. 项目管理/Bug管理/问题管理
2. DNS -> Domain Name System
3. LB(负载均衡) Load Balance
4. CDN Content Delivery Network 内容分发网络
5. RPC框架 Remote Procedure Call 远程过程调用
6. 名字发现/服务发现 etcd
7. 关系数据库
8. NoSQL
9. 消息中间件
10. 代码管理
11. 持续集成
12. 日志系统
13. 监控系统
14. 配置系统
15. 发布系统/部署系统
16. 跳板机
17. 机器管理

##### 创业公司的选择

1. 选择合适的语言
2. 选择合适的组件和云服务商
3. 制定流程和规范
4. 自研和选型合适的辅助系统
5. 选择过程中需要思考的问题