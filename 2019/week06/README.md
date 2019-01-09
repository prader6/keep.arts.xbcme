## ARTS(week06)

### 算法题(Algorithm)

[两个数组的交集 II](https://github.com/geekwho11/learn.leetcode.xbcme/tree/master/php/src/350.intersection-of-two-arrays-ii)

### 阅读点评(Review)

#### [Don't Use Apache Kafka Consumer Groups the Wrong Way!](https://dzone.com/articles/dont-use-apache-kafka-consumer-groups-the-wrong-wa)

#### 阅读笔记

Apache Kafka非常好。如果你打算使用它，你需要非常小心。这里有一些常见的"坑"。

##### 消费组的经验

1. 消费者隶属于一个消费组，意味着多个消费者是竞争模式，根据主题消息的分区和成员分布会发生竞争。
2. 将消费者当作不同消费组的话，就意味着提供发布订阅模式，主题分区的消息会发送给所有组的消费者。

消费组还有一个高级特性就是自动平衡。当一个消费者加入到一个消费组的时候，还没有达到一个分区一个消费者的限制，那么自动平衡会重新分配分区到不同的消费者。同样的，一个消费者下线了，分区也会被重新分配给剩下的消费者。

到目前为止，我所说的通过[KafkaConsumer](https://kafka.apache.org/0110/javadoc/index.html?org/apache/kafka/clients/consumer/KafkaConsumer.html) API的subscribe()方法，确实如此。这个方法强制要求一个消费者必须通过group.id设置一个消费组，因为分区可能会被重新分配。消费者不能决定自己想要读取的分区。通常情况下，第一个加入的消费者会在其他消费者加入组的时候，进行分区的分配。

如果你同时使用subscribe()、assign()、手动分配的话，那么坑就来了。

使用assign()方式不恰当的话，会让你的消费者在同一个消费组，变成了发布订阅模式。

什么是补偿提交？

__consumer_offsets的消息结构如下：

- key = [group, topic, partition]
- value = offset

那么在补偿提交时，有一个消费者挂掉了，然后重新启动后，可能会丢消息。

### 技术技巧(Tip)

#### Composer 自动加载

```
psr4
{
    "autoload": {
        "psr-4": {
            "Monolog\\": "src/", 
            "Vendor\\Namespace\\": ""
        }
    }
}
```

项目目录以root标识，composer目录以path标识

| 全局命名空间                        | 命名前缀                      | 基础目录              | 文件解析路径                     | 文件命名空间                            |
| ----------------------------- | ------------------------- | ----------------- | -------------------------- | --------------------------------- |
| \Monolog\Base                 | ```Monolog\\```           | {path}/src/       | {path}/src/Base.php        | namespace Monolog                 |
| \Monolog\Logger\Base          | ```Monolog\\```           | {path}/src/Logger | {path}/src/Logger/Base.php | namespace Monolog\Logger          |
| \Vendor\Namespace\Base        | ```Vendor\\Namespace\\``` | {root}/           | {root}/Base.php            | namespace Vendor\Namespace        |
| \Vendor\Namespace\Logger\Base | ```Vendor\\Namespace\\``` | {root}/Logger     | {root}/Logger/Base.php     | namespace Vendor\Namespace\Logger |

多个目录进行搜索

```
{
    "autoload": {
        "psr-4": { "Monolog\\": ["src/", "lib/"] }
    }
}
```

| 全局命名空间               | 命名前缀            | 基础目录              | 文件解析路径                     | 文件中命名空间                  |
| -------------------- | --------------- | ----------------- | -------------------------- | ------------------------ |
| \Monolog\Base        | ```Monolog\\``` | {path}/src/       | {path}/src/Base.php        | namespace Monolog        |
|                      |                 | {path}/lib/       | {path}/lib/Base.php        |                          |
| \Monolog\Logger\Base | ```Monolog\\``` | {path}/src/Logger | {path}/src/Logger/Base.php | namespace Monolog\Logger |
|                      |                 | {path}/lib/Logger | {path}/lib/Logger/Base.php |                          |

如果在默认的路径下，找不到相关的类，可以在指定目录进行搜索。

```
{
    "autoload": {
        "psr-4": { "": "src/" }
    }
}
```

| 全局命名空间        | 命名前缀            | 基础目录        | 文件解析路径                      | 文件命名空间            |
| ------------- | --------------- | ----------- | --------------------------- | ----------------- |
| \Noexist\Base | ```Noexist\\``` | {path}/src/ | {path}/src/Noexist/Base.php | namespace Noexist |

#### 参考链接

1. [composer autoload](https://getcomposer.org/doc/04-schema.md#autoload)

### 分享(Share)

#### [远离温水煮青蛙，新的一年做个有规划的技术人](https://dbaplus.cn/news-149-1957-1.html)

#### 阅读笔记

1. 远离温水煮青蛙的状态
2. 长期规划，短期规划
3. 被动变为主动
4. 共同成长
5. 充分利用碎片时间
6. 读书的误区

#### 思考总结

看到一年前的分享，感触颇深。

知易行难。

知道，并不能让你成长，只有做到，才能真正让你有成长。

践行才是你真正的刚需。