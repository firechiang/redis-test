#### [一、Windows单节点开发搭建][1]
#### [二、Centos单节点开发搭建][2]
#### [三、Centos单节点生产搭建][3]


#### 客户端简单使用
```bash
$ redis-cli -h 127.0.0.1 -p 6379  # 连接redis(如果是连接有密码的redis，在连接成功后，输入 auth，再输入密码即可)
$ help                            # 查看帮助
$ help set                        # 查看set命令的简单使用帮助(其它帮助命令同理)
$ help @set                       # 查看set命令的详细使用帮助(其它帮助命令同理)

$ flushdb                         # 清空当前数据库的数据
$ flushall                        # 清空所有数据库的数据

$ type 'key'                      # 显示某个key的数据类型
$ OBJECT encodeing 'key'          # 编码某个key的数据，显示原始数据类型

$ getstr name a                   # 设置新值，再返回原有的值

$ append name sasas               # 追加数据

$ settrange name 0 2 dd           # 将dd设置到0到2之间
$ getrange name 0 2               # 获取字符串0到2的数据(相当于截取字符串) 

$ mset name1 a name2 b            # 一次设置多个值(注意：mset具有原子性，要么全部成功要么全部失败)
$ mget name1 name2                # 一次取多个值

$ incr 'key'                      # 递增1
$ decr 'key'                      # 递减1

$ setbit name 1 1                 # 设置ASCII，第一个1是偏移量，第二个1是值
$ get name                        # 看看ASCII的值是啥

$ incrby name 2                   # 递增指定值(可负数就是减) 
$ incrbyfloat name 2.2            # 递增指定带小数点的值 

$ keys *                          # 查找所有key
$ keys ?                          # 查找只有一个字符的key(注意：写几个?号代表查找几个字符的key)

$ strlen 'key'                    # 显示某个key的数据长度(适用于string类型数据)
```



[1]: https://github.com/firechiang/redis-test/tree/master/docs/windows-single-node-install-dev.md
[2]: https://github.com/firechiang/redis-test/tree/master/docs/centos-single-node-install-dev.md
[3]: https://github.com/firechiang/redis-test/tree/master/docs/centos-single-node-install-prod.md
