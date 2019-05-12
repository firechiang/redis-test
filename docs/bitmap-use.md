#### 一、BitMap优势，我们知道1个Byte占1个字节，1个字节占8位(bit)，所以BitMap本身会极大的节省储存空间(直接操控二进制)

#### 二、setbit命令(SETBIT key offset value)，offset偏移量(下标)，value值(只能是0或者1)
```bash
$ setbit k1 1 1                                  # 返回原来的值
$ getbit k1 1                                    # 获取偏移量(下标)为1的值
$ bitpos k1 0 1 1                                # 获取值为0的数据的偏移量(说的是第一次值为0)，后面的两个1是起始范围(起始范围是已字节为单位，那这里的起始范围就是0-7，就是第一个字节到第一个字节)
$ bitcount k1 0 1                                # 统计值为1的数量，后面的0 1个区间范围(起始范围是已字节为单位，那这里的起始范围就是0-15，就是第一个字节到第二个字节)

# bit运算(注意：计算时较短的bit位自动填充0，空key也会用0填满所有bit位)
$ bitop and destKey key [key...]                 # 对一个过多个key求逻辑并，并将结果保存到destKey
$ bitop or destKey key [key...]                  # 对一个过多个key求逻辑或，并将结果保存到destKey
$ bitop xor destKey key [key...]                 # 对一个过多个key求逻辑异或，并将结果保存到destKey
$ bitop not destKey key                          # 对给定的key求逻辑非，并将结果保存到destKey
```

#### 三、使用场景
##### 3.1、用户签到(思路：以用户ID+年为key，当前日期是当前年的第几天为偏移量。也可以使用以用户ID为key，((今天的日期)-(开始有签到功能的日期)) / 86400为偏移量)
```bash
int dayOfYear = LocalDate.now().getDayOfYear();
$ setbit key dayOfYear 1                         # 返回0说明签到成功，返回1说明已签过到
```

##### 3.2、统计活跃用户(思路：使用时间作为cacheKey，然后用户ID为offset，如果当日活跃过就设置为1)
```bash
# 计算1月10号到12的活跃用户
$ bitop or stat 2019-01-10 2019-01-11 2019-01-12 # 将这些日期的数据放到另一个BitMap里面
$ bitcount stat 0 -1                             # 统计值为1的数量
```