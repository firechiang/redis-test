#### [一、Windows单节点开发搭建][1]
#### [二、Redis4.X单节点搭建][2]
#### [三、Redis5.X单节点搭建][3]
#### [四、Redis4.X集群搭建][11]
#### [五、Redis5.X集群搭建][12]
#### [六、简单数据类型简单使用][5]
#### [七、list数据类型简单使用][6]
#### [八、hash数据类型简单使用][7]
#### [九、set数据类型简单使用][8]
#### [十、sortedSet数据类型简单使用][9]
#### [十一、位图BitMap使用以及使用场景][10]
#### 十二、客户端简单使用
```bash
$ redis-cli -h 127.0.0.1 -p 6379        # 连接redis(如果是连接有密码的redis，在连接成功后，输入 auth，再输入密码即可)
$ help                                  # 查看帮助
$ help set                              # 查看set命令的简单使用帮助(其它帮助命令同理)
$ help @set                             # 查看set命令的详细使用帮助(其它帮助命令同理)

$ flushdb                               # 清空当前数据库的数据
$ flushall                              # 清空所有数据库的数据

$ keys *                                # 查找所有key
$ keys ?                                # 查找只有一个字符的key(注意：写几个?号代表查找几个字符的key)

$ type 'key'                            # 显示某个key的数据类型
$ OBJECT encodeing 'key'                # 编码某个key的数据，显示原始数据类型

$ service redis start                   # 启动redis
$ service redis stop                    # 停止redis(注意：如果redis有设置密码，该命令无法停止redis)
$ redis-cli -p 6379 -a jiang shutdown   # 停止redis(用登录密码连接redis然后执行shutdown停止redis)
$ sudo chkconfig redis on               # 设置redis开机启动
$ sudo chkconfig redis off              # 关闭redis开机启动

$ config get *                          # 获取Redis的配置信息
$ config set dbfilename dump.rdb        # 设置Redis的dbfilename配置信息
```

#### 十三、Redis集群分片
##### 13.1，集群将整个数据库分为16384个slot(槽位)，数据最终所在slot(槽位)的计算公式是：slot_number = crc16(key) % 16384
##### 13.2，上面的crc16是16位的[循环冗余检验码](https://baike.baidu.com/item/%E5%BE%AA%E7%8E%AF%E5%86%97%E4%BD%99%E6%A0%A1%E9%AA%8C%E7%A0%81)函数，简单可以理解为计算哈希
##### 13.3，假如集群有四个节点 7000，7001，7002，7003。那么它们会平均分配16384个slot(槽位)，7000负责0-5060的slot(槽位)数据，7001负责5461-10022的slot(槽位)数据，7002负责10923-16383的slot(槽位)数据，7003负责5061-5460和10023-10922的slot(槽位)数据。
##### 13.4，当有新的节点加如集群，只需要从旧节点里面分配一些slot(槽位)给新的节点即可
```bash

```
#### 十四、AOF持久化
##### 14.1、AOF持久化优点：默认fsync每秒执行，性能很好，最多丢失一秒的数据，自动优化AOF文件(删除重复命令)
##### 14.2、AOF持久化缺点：AOF文件过大，恢复较慢(试想一下我的服务器5年没有停过机，现在突然停机了，5年啊，得有多少命令啊，重新执行一遍。当然AOF默认会合并一些命令操作，尽量减少记录命令)
##### 14.3、如果AOF重写的时候出现故障导致命令写半截，可以使用redis-check-aop工具修复
##### 14.4、AOF写入磁盘策略(appendfsync 配置文件这个选项的值可以是always、everysec、no)
```bash
Always: 服务器每写入一条命令，就调用一次fdatasync，将缓冲区里面的命令写入磁盘。这种模式下，服务器出现故障，基本也不会丢失任何已经成功执行的命令数据。
Everysec(默认，推荐使用): 服务器每一秒调用一次fdatasync，将缓冲区里面的命令写入磁盘，这种模式下，服务器出现故障，最多丢失一秒内的执行的命令数据。
No: 服务器不主动调用fdatasync，由系统决定何时将缓冲区的命令写入磁盘，这种模式下，服务器遭遇以外停机时，丢失命令的数据时不确定的。
三者的运行速度: always的速度最慢，everysec和no都很快。
```
##### 14.5、AOF重写(优化替换原来的AOF持久化文件)的触发方式，手动: 客户端向服务器发送bgrewriteaop命令。自动: 在配置文件里面配置如下参数
```bash
auto-aof-rewrite-min-size 64M   # 触发AOF重写所需的最小体积，只有AOF文件体积大于等于这个值时才会考虑是否进行AOF重写(这个选项用于避免过小的AOF文件进行重写)
auto-aof-rewrite-percentage 100 # 这个配置的意思是当AOF文件的增量大于起始auto-aof-rewrite-min-size的100%时(就是文件大小翻了一倍)，才会触发AOF重写
(如果服务器刚启动不久，还没有进行过AOF重写，那么使用服务器启动时载入的AOF文件的体积来作为基准值)，将auto-aof-rewrite-percentage设为0表示关闭自动重写AOF

appendonly no                   # 是否开启AOF重写，默认就是这个no，要开启AOF的话改为yes，RDB将停止
```

#### 十五、RDB持久化
##### 15.1、自动触发RDB持久化，是redis持久化默认的方式，按照配置文件中的条件满足就执行bgsave(非阻塞方式)
##### 15.2、redis可以手动触发RDB持久化，客户端发起save(阻塞方式)，bgsave(非阻塞方式)命令即可
##### 15.3、save阻塞方式的优势：保证数据不丢失。缺点：阻塞期间将无法给客户端提供服务，这个太严重了，相当于服务器死机了
##### 15.4、bgsave非阻塞方式的优势：持续为客户端提供服务。缺点：可能会丢失持久化期间的数据(在持久化的时候，客户端同时也在写入，这个时候服务器宕机了，那么这个时候客户端写入的数据就丢了)

[1]: https://github.com/MicrosoftArchive/redis/releases
[2]: https://github.com/firechiang/redis-test/tree/master/docs/centos-single-node-install-dev.md
[3]: https://github.com/firechiang/redis-test/tree/master/docs/centos-single-node-install-prod.md
[5]: https://github.com/firechiang/redis-test/tree/master/docs/string-use.md
[6]: https://github.com/firechiang/redis-test/tree/master/docs/list-use.md
[7]: https://github.com/firechiang/redis-test/tree/master/docs/hash-use.md
[8]: https://github.com/firechiang/redis-test/tree/master/docs/set-use.md
[9]: https://github.com/firechiang/redis-test/tree/master/docs/sortedset-use.md
[10]: https://github.com/firechiang/redis-test/tree/master/docs/bitmap-use.md
[11]: https://github.com/firechiang/redis-test/tree/master/docs/redis4-cluster-node-install-prod.md
[12]: https://github.com/firechiang/redis-test/tree/master/docs/redis5-single-node-install-prod.md
