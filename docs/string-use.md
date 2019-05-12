#### 简单数据类型简单使用
```bash
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

$ strlen 'key'                    # 显示某个key的数据长度(适用于string类型数据)
```