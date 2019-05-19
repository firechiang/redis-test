#### 一、环境准备
```bash
$ sudo mkdir -p /usr/redis-4.0.14                            # 创建redis安装目录
$ wget -P /usr/redis-4.0.14 http://rubygems.org/downloads/redis-4.1.1.gem
$ sudo yum install -y gcc                                    # 安装gcc编译环境
$ wget http://download.redis.io/releases/redis-4.0.14.tar.gz # 下载安装包
$ sudo tar -zxvf redis-4.0.14.tar.gz -C ./                   # 解压到当前目录
```

#### 二、编译安装以及测试是否安装成功
```bash
$ cd redis-4.0.14                                            # 跳转到redis解压目录下
$ sudo make & make PREFIX=/usr/redis-4.0.14 install          # 编译，安装到 /usr/redis-4.0.14 目录
$ sudo scp -r redis.conf utils /usr/redis-4.0.14             # 拷贝配置文件和工具脚本到 /usr/redis-4.0.14 目录
$ sudo scp -r ./src/redis-trib.rb /usr/redis-4.0.14/bin      # 拷贝集群管理工具到 /usr/redis-4.0.14/bin 目录
$ cd /usr/redis-4.0.14/bin                                   # 到redis安装目录(/usr/redis-4.0.14/bin)

$ ./redis-server                                             # 启动redis，测试是否安装成功
$ ps -aux | grep redis                                       # 查看redis进程
$ redis-cli shutdown                                         # 停止redis
```

#### 三、修改[vi /usr/redis-4.0.14/redis.conf]配置文件(配置项都存在，只需要修改值即可)
```bash
bind 0.0.0.0                                                 # 绑定IP(允许那些IP可以访问，0.0.0.0是所有IP都可以访问)
port 6379                                                    # 端口
daemonize yes                                                # 以后台进程方式启动redis
requirepass jiang                                            # 配置登录密码是jiang
protected-mode yes                                           # 开启保护模式
dir /usr/redis-4.0.14/data                                   # 持久化数据所在目录(注意：手动创建目录)
logfile "/usr/redis-4.0.14/log/redis-server.log"             # 日志所在目录(注意：手动创建目录和文件)

cluster-enabled yes                                          # 打开集群模式
cluster-config-file /usr/redis-4.0.14/cluster-nodes.conf     # 集群信息文件(这个文件是Redis集群自动生成的)
cluster-node-timeout 10000                                   # 集群节点超时时间
cluster-slave-validity-factor 5                              # 切换为主机的时间(0-10，值越小说明检查越频繁)
repl-ping-slave-period 5                                     # 检查主节点健康状态                        
repl-timeout 30                                              # 主从复制超时时间，该值要大于 repl-ping-slave-period 的值
```

#### 四、修改Redis启动脚本[vi /usr/redis-4.0.14/utils/redis_init_script]
```bash
EXEC=/usr/redis-4.0.14/bin/redis-server                      # redis服务脚本所在目录
CLIEXEC=/usr/redis-4.0.14/bin/redis-cli                      # redis客户端脚本所在目录
CONF="/usr/redis-4.0.14/redis.conf"                          # redis配置文件在目录(注意：这个配置是带"双引号"的)

$CLIEXEC -p $REDISPORT -a jiang shutdown                     # 关闭redis时所使用的代码，加上 -a jiang(就是Redis密码)
```

#### 五、分发Redis安装文件到各个节点
```bash
$ scp -r /usr/redis-4.0.14 root@server002:/usr/redis-4.0.14
$ scp -r /usr/redis-4.0.14 root@server003:/usr/redis-4.0.14
$ scp -r /usr/redis-4.0.14 root@server004:/usr/redis-4.0.14
$ scp -r /usr/redis-4.0.14 root@server005:/usr/redis-4.0.14
$ scp -r /usr/redis-4.0.14 root@server006:/usr/redis-4.0.14
```

#### 六、修改[vi /usr/redis-4.0.14/redis.conf]各个节点上的端口号，从7000-7005
```bash
port 7000                                                    # 注意：将7000修改成当前机器Redis的端口
```

#### 七、修改[vi /usr/redis-4.0.14/utils/redis_init_script]各个节点上的端口号，从7000-7005
```bash
$CLIEXEC -p 7000 -a jiang shutdown                           # 将 $CLIEXEC -p $REDISPORT -a jiang shutdown 替换成 $CLIEXEC -p 7000 -a jiang shutdown
```

#### 八、安装Rubby环境(集群所有节点都要安装，集群控制工具依赖环境)
```bash
$ sudo yum install -y gcc                                    # 安装gcc编译器，如果没有安装就安装一下
$ wget -P /home/tools https://cache.ruby-lang.org/pub/ruby/2.6/ruby-2.6.3.tar.gz
$ cd /home/tools
$ tar -zxvf ruby-2.6.3.tar.gz -C ./
$ cd ruby-2.6.3
$ sudo ./configure --prefix=/opt/ruby-2.6.3
$ sudo make && make install
$ sudo ln -s /opt/ruby-2.6.3/bin/ruby /usr/bin/ruby
$ sudo ln -s /opt/ruby-2.6.3/bin/gem /usr/bin/gem
$ ruby -v                                                    # 验证ruby是否安装成功
$ yum -y install zlib-devel
$ cd /home/tools/ruby-2.6.3/ext/zlib
$ ruby ./extconf.rb
$ vi /home/tools/ruby-2.6.3/ext/zlib/Makefile                # 将 zlib.o: $(top_srcdir)/include/ruby.h 替换成 zlib.o: ../../include/ruby.h
$ sudo make && make install
$ gem install --local /usr/redis-4.0.14/redis-4.1.1.gem      # 安装redis集群控制依赖(redis-4.1.1.gem文件我们在第一步已经下载好了)
```

#### 九、修改集群控制工具的配置文件[vi /opt/ruby-2.6.3/lib/ruby/gems/2.6.0/gems/redis-4.1.1/lib/redis/client.rb]为其指定Redis密码，集群所有节点都要修改(如果不知道client.rb文件在哪里，可使用该命令查找：find / -name 'client.rb')
```bash
:password => "jiang"
```

#### 十、配置Redis开机启动(集群所有节点都要配置)
##### 10.1、复制Redis启动脚本到 /etc/init.d/redis
```bash
$ sudo cp /usr/redis-4.0.14/utils/redis_init_script /etc/init.d/redis
```

##### 10.2、启动集群所有节点的Redis
```bash
$ service redis start                                        # 启动redis
$ service redis stop                                         # 停止redis(注意：如果redis有设置密码，该命令无法停止redis)
$ sudo chkconfig redis on                                    # 设置redis开机启动
$ sudo chkconfig redis off                                   # 关闭redis开机启动
```

#### 十一、任选一台机器，创建集群(注意：集群最少三个主节点，创建节点时要使用IP否则可能会报：ERR Invalid node address specified 错误)
```bash
$ cd /usr/redis-4.0.14/bin
# --replicas 1 是指定副本数量，1就是指每一个主节点，都有一个从节点，现在共6台机器，那就是3主3备(主从分配规则: 按照创建命令的填写顺序，先创建主节点，再创建从节点)
$ ./redis-trib.rb create --replicas 1 192.168.229.133:7000 192.168.229.129:7001 192.168.229.134:7002 192.168.229.132:7003 192.168.229.137:7004 192.168.229.138:7005
```

#### 十二、连接测试集群
```bash
$ cd /usr/redis-4.0.14/bin
$ ./redis-cli -c -p 7000 -a jiang                            # -c表示集群模式连接
$ set k1 k1
$ get k1 k1
```


#### 十三、删除redis安装包和解压目录(看情况而定)

