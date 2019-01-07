## ARTS(week02)

### 算法题(Algorithm)

[买卖股票的最佳时机 II](https://github.com/geekwho11/learn.leetcode.xbcme/blob/master/php/src/122.best-time-to-buy-and-sell-stock-ii)

### 阅读点评(Review)

#### [How To Add Swap Space on Ubuntu 18.04](https://www.digitalocean.com/community/tutorials/how-to-add-swap-space-on-ubuntu-18-04)

#### 阅读笔记

##### 什么是交换区？

交换区是一个系统文件，被设计用于当系统内存不足，写入额外临时数据的地方。

##### 步骤

文章讲述了如何添加交换区大小的步骤

1. 检查系统交换区信息
2. 检查硬盘的可用空间
3. 创建一个交换区文件
4. 启用交换区文件
5. 永久启用交换区文件
6. 调优交换区设置

##### 常用命令

1.
```
   sudo swapon -s 查看交换区概要

   free -m 显示内存使用信息

   sudo swapon --show 显示已定义表格的交换区
   free -h 以友好人类可读的方式显示内存使用情况
   df -h 以友好人类可读的方式显示文件系统硬盘使用情况
   sudo fallocate -l 1G /swapfile 创建1GB的文件
   ls -lh /swapfile 查看交换区文件
   sudo chmod 600 /swapfile 设置文件权限为600
   sudo mkswap /swapfile 将文件格式化为swap格式
   sudo swapon /swapfile 启用交换区文件
   sudo cp /etc/fstab /etc/fstab.bak 备份fstab
   echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab 将交换区信息写入文件系统配置文件
```
##### 调优

swappiness
1. ```
   cat /proc/sys/vm/swappiness  查看当前swappiness参数的值
   ```
2. 参数为0时，系统会尽量不写入数据到交换分区，除非内存全部占用。

3. 参数为100时，系统会尽可能的写入更多的数据到交换分区，尽量让内存RAM空余。

4. ```
   sudo sysctl vm.swappiness=10 通过命令设置该参数的值
   sudo nano /etc/sysctl.conf 永久生效需要写入配置文件
   vm.swappiness=10
   ```
   vfs_cache_pressure
5. ```
   cat /proc/sys/vm/vfs_cache_pressure 查看系统当前的值
   ```
6. 这个参数是决定你的系统缓存系统节点和目录结构的优先级。
7. ```
   sudo sysctl vm.vfs_cache_pressure=50
   sudo nano /etc/sysctl.conf
   vm.vfs_cache_pressure=50
   ```
### 技术技巧(Tip)

##### 如何调整swap大小

1. Boot from a LiveCD/DVD/USB
2. sudo swapoff -a
3. Unmount swap partion
4. Resize
5. Find the new UUID of the two partitions
6. sudo blkid
7. 如果swap的uuid有改变的话，就需要修改新的uuid。

#### 参考链接

1. [Cannot move swap space](https://askubuntu.com/questions/510393/cannot-move-swap-space)

### 分享(Share)

#### [职场分身术：从给答案到做引导](https://time.geekbang.org/column/article/802)

#### 思考总结

当初从技术转到管理时，也会遇到这样的问题，每天问你问题的人会非常多，会显得你很忙，但有很多更重要更紧急的事情，就没有时间去做了。

如果问题是和业务相关的话，可能需要整理总结出相关的文档

如果是技术的话，可以让新人多尝试，多花点时间分析，了解问题的根本原因。

慢慢，整理一套带新人的流程后，会比较顺利很多。不光自己得到了成长，新人也有进步。


