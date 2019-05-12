#### 一、hash数据类型简单使用(注意：一个hash中最多包含2^32-1和键值对)
```bash
$ hset k1 name jiang age 1
$ hmset k1 phone 131434562 type 1
$ hget k1 name
$ hkeys k1                          # 查看k1所有的属性
$ hmget k1 name age                 # 查看多个属性的值
$ hvals k1                          # 查看k1所有属性的值
$ hlen k1                           # 查看k1属性的个数 
$ hgetall k1                        # 查看所有的键值对
$ hdel k1 name                      # 删除某个属性
$ hincrby k1 age 1                  # 对某个属性进行整型原子递增(写负数，就是减)
$ hincrbyfloat k1 age 1.2           # 对某个属性进行小数原子递增(写负数，就是减)
```

#### 二、使用场景
##### 2.1、微博好友关注(思路：用户ID为key，好友ID为field，value为关注时间)
##### 2.2、用户维度统计，比如关注数，粉丝数(思路：用户ID为key，不同维度为field，value为统计数)