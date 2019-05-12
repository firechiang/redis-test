#### 一、sortedSet数据类型简单使用(注意：有序集合sortedSet里面每一个元素关联着一个浮点分值(score)，并按照分值从小到大的顺序排列集合中的元素，分值可以相同，sortedSet最多包含2^32-1个元素)
```bash
$ zadd k1 1 name 2 age                               # 添加数据，数字是score，就是排序用的(如果元素已存在，则使用新的score)
$ zrange k1 0 -1                                     # 按照范围查看数据(0 -1就是全部数据)
$ zrange k1 0 -1 withscores                          # 按照范围查看数据以及score分值(0 -1就是全部数据，先大再小排序)
$ zrange k1 -3 -1 withscores                         # 按照范围查看数据以及score分值(0 -1就是全部数据，先小再大排序)
$ zscore k1 name                                     # 根据元素显示对应的score分值
$ zincrby k1 1 name                                  # 原子增加对应分值，用于调整排序(负数就是减)      
$ zrank k1 name                                      # 返回元素正序后的下标
$ zrevrank k1 name                                   # 返回元素逆序后的下标
$ zrangebyscore k1 0 2                               # 按照score的区间返回数据
$ zrangebyscore k1 0 2 withscores                    # 按照score的区间返回数据以及score信息
$ zrangebyscore k1 0 2 limit 0 1                     # 按照score的区间返回数据，并使用limit
$ zrangebyscore k1 0 2 withscores limit 0 1          # 按照score的区间返回数据以及score信息，并使用limit
$ zrangebyscore k1 -inf +inf                         # 查看所有数据
$ zrangebyscore k1 -inf +inf limit 0 1               # 查看所有数据，并使用limit
$ zrangebyscore k1 -inf +inf withscores              # 查看所有数据以及score信息
$ zrangebyscore k1 -inf +inf withscores limit 0 1    # 查看所有数据以及score信息，并使用limit
$ zremrangebyrank k1 0 1                             # 移除排序后下标范围在0 1的范围的元素
$ zcard k1                                           # 查看元素的个数
$ zcount k1 0 2                                      # 查看指定score范围的元素的个数
$ zunionstore k3 2 k1 k2                             # 取k1 k2的并集到新的集合k3当中，返回k3集合元素的个数(2的意思就是取几个集合的并集，我们取的时k1和k2所以就是2)
$ zunionstore k3 2 k1 k2 aggregate sum               # 取k1 k2的并集到新的集合k3当中，score分值以聚合的方式，返回k3集合元素的个数(2的意思就是取几个集合的并集，我们取的时k1和k2所以就是2)
$ zunionstore k3 2 k1 k2 weights 1 0.5 aggregate sum # 取k1 k2的并集到新的集合k3当中，score分值以聚合的方式，返回k3集合元素的个数(2的意思就是取几个集合的并集，我们取的时k1和k2所以就是2)(k1的score分值乘以1，k1的score分值乘以0.5)
```

#### 二、使用场景
##### 2.1、网易云音乐歌曲排名(思路：排行版为key，歌曲为元素，那首歌听的多，就将其score分值加大，它就排在前面了)
##### 2.2、微博翻页(思路：栏目为key，微博为元素，微博发布时间戳为score分值，这样默认最新发布的微博就在最前面了)