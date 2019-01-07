## ARTS(week01)

### 算法题(Algorithm)

[删除排序数组中的重复项](https://github.com/geekwho11/learn.leetcode.xbcme/tree/master/php/src/26.remove-duplicates-from-sorted-array)

### 阅读点评(Review)

#### [Starting Off With a Sha-Bang](https://www.tldp.org/LDP/abs/html/sha-bang.html)

#### 阅读笔记

1. 最简单的例子，一个脚本只是把一些系统内置的命令列表存储在文件里。

2. *sha-bang* (#!)在一个脚本的顶部，告诉你的系统解释器需要提前加载的命令集。

3. ```
   #!实际上是2个字节的魔法数字。一个特殊文件类型的标记或者在这个情况下，可执行shell脚本。
   ```

4. 请注意，sha-bang的路径一定要是争取的，否则就会出现错误信息Command not found.

5. ```
   #!可以省略，如果脚本只是常用的系统内置命令，没有用到内置shell标识。
   ```

6. 我就是给了一个错误的路径导致出现奇怪的错误信息。

#### 思考总结

##### 什么是Sha-Bang

1. 默认Bash 脚本，是不需要Sha-Bang。一个脚本可以直接执行相关的Linux命令，不需要Sha-Bang。

2. ```
   #!/bin/bash 这是一个Bash Script脚本正确的文件头。
   ```

##### 为什么要用Sha-Bang
1. 主要目的是方便系统根据文件头去加载解析器。
2. [wiki Shebang](https://zh.wikipedia.org/wiki/Shebang)

##### 怎么用

```
下面列出了一些典型的 shebang 解释器指令：

#!/bin/sh—使用 sh，即 Bourne shell 或其它兼容 shell 执行脚本
#!/bin/csh—使用 csh，即 C shell 执行
#!/usr/bin/perl -w—使用带警告的 Perl 执行
#!/usr/bin/python -O—使用具有代码优化的 Python 执行
#!/usr/bin/php—使用 PHP 的命令行解释器执行
```

### 技术技巧(Tip)
在Dockerfile里EXPOSE 只是指定打算主机的端口，不会自动进行映射。
在docker run -P时，会自动进行映射，容器端口为随机端口，宿主端口为设定的端口号。

在docker run -p <宿主端口>:<容器端口> 明确指定映射的端口。

#### 参考链接

1. [EXPOSE 声明端口](https://yeasy.gitbooks.io/docker_practice/image/dockerfile/expose.html)
2. [Expose vs publish: Docker port commands explained simply](https://medium.freecodecamp.org/expose-vs-publish-docker-port-commands-explained-simply-434593dbc9a3)

### 分享(Share)

#### [如何从错误中成长](https://time.geekbang.org/column/article/2795)

#### 阅读笔记

文章总结工作中可能会遇到的四大类错误：

1. 第一类，伸展错误
2. 第二类，无知错误
3. 第三类，粗心错误
4. 第四类，高风险错误

我们需要避免第二类和第三类错误，针对第二类错误，需要经常复盘总结，避免下次犯同样的错误。第三类错误，建立添加检查列表去确认，避免遗漏导致出错。

避免犯错的方法论

1. 为了避免伸展错误，尽可能地提供一些培训机制。
2. 为了避免无知错误，要做好信息的透明和共享。
3. 为了避免粗心错误，设置一定的复盘机制。
4. 为了避免高风险错误，所有的决定尽可能都有一个备用方案。


