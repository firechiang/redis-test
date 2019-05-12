#### [一、Windows单节点开发搭建][1]
#### [二、Centos单节点开发搭建][2]
#### [三、Centos单节点生产搭建][3]


#### 四、客户端简单使用
```bash
$ redis-cli -h 127.0.0.1 -p 6379  # 连接redis(如果是连接有密码的redis，在连接成功后，输入 auth，再输入密码即可)
$ help                            # 查看帮助
$ help set                        # 查看set命令的简单使用帮助(其它帮助命令同理)
$ help @set                       # 查看set命令的详细使用帮助(其它帮助命令同理)

$ flushdb                         # 清空当前数据库的数据
$ flushall                        # 清空所有数据库的数据

$ keys *                          # 查找所有key
$ keys ?                          # 查找只有一个字符的key(注意：写几个?号代表查找几个字符的key)

$ type 'key'                      # 显示某个key的数据类型
$ OBJECT encodeing 'key'          # 编码某个key的数据，显示原始数据类型
```

#### [五、简单数据类型简单使用][5]
#### [六、list数据类型简单使用][6]
#### [七、hash数据类型简单使用][7]
#### [八、set数据类型简单使用][8]
#### [九、sortedSet数据类型简单使用][9]
#### [十、位图BitMap使用以及使用场景][10]

[1]: https://github.com/MicrosoftArchive/redis/releases
[2]: https://github.com/firechiang/redis-test/tree/master/docs/centos-single-node-install-dev.md
[3]: https://github.com/firechiang/redis-test/tree/master/docs/centos-single-node-install-prod.md
[5]: https://github.com/firechiang/redis-test/tree/master/docs/string-use.md
[6]: https://github.com/firechiang/redis-test/tree/master/docs/list-use.md
[7]: https://github.com/firechiang/redis-test/tree/master/docs/hash-use.md
[8]: https://github.com/firechiang/redis-test/tree/master/docs/set-use.md
[9]: https://github.com/firechiang/redis-test/tree/master/docs/sortedset-use.md
[10]: https://github.com/firechiang/redis-test/tree/master/docs/bitmap-use.md
