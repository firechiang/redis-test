#### 一、set数据类型简单使用(注意：set是无序去重的，最多包含2^32-1个元素)
```bash
$ sadd k1 q w e r           # 添加数据 
$ srem k1 w e               # 删除值为w和e的元素
$ smembers k1               # 查看所有数据(注意：在生成环境尽量不要遍历大的set集合，以免redis阻塞)
$ sismember k1 r            # 判断r元素是否存在(0不存在，1存在)
$ srandmember k1 1          # 随机返回几个元素(1代表1个)
$ spop k1 1                 # 随机弹出几个元素(1代表1个)，可用于抽奖等业务
$ scard k1                  # 返回元素的个数
$ sunion k1 k2              # 返回聚合后的两个set
$ sinter k1 k2              # 取两个set都存在的元素
$ sinterstore key1  k1 k2   # 将两个set都存在的元素，放到一个新的set里面，名字叫key1
```

#### 二、使用场景
##### 2.1、微博的共同关注(思路：将每个用户关注的用户放在集合中，求交集即可)
