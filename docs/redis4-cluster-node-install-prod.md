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
cluster-node-timeout 15000                                   # 集群节点超时时间
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

#### 七、安装Rubby环境(集群所有节点都要安装，集群控制工具依赖环境)
```bash
$ wget -P /home/tools https://cache.ruby-lang.org/pub/ruby/2.6/ruby-2.6.3.tar.gz
$ cd /home/tools
$ tar -zxvf ruby-2.6.3.tar.gz -C ./
$ cd ruby-2.6.3
$ sudo mkdir -p /opt/ruby-2.6.3
$ ./configure --prefix=/opt/ruby-2.6.3
$ sudo make && make install
$ ln -s /opt/ruby-2.6.3/bin/ruby /usr/bin/ruby
$ ln -s /opt/ruby-2.6.3/bin/gem /usr/bin/gem
$ yum install -y ruby rubygems
$ gem install --local /usr/redis-4.0.14/redis-4.1.1.gem      # 安装redis集群控制依赖(redis-4.1.1.gem文件我们在第一步已经下载好了)
```

#### 七、修改集群控制工具的配置文件[vi /usr/local/share/gems/gems/redis-3.3.0/lib/redis/client.rb]为其指定Redis密码，集群所有节点都要修改(如果不知道client.rb文件在哪里，可使用该命令查找：find / -name 'client.rb')
```bash
:password => "jiang"
```

#### 八、配置环境变量[vi ~/.bashrc]，(集群所有节点都要配)
```bash
export REDIS_HOME=/usr/redis-4.0.14
PATH=$PATH:$REDIS_HOME/bin                                   # linux以 : 号隔开，windows以 ; 号隔开

$ source ~/.bashrc                                           # （系统重读配置）在各个机器上执行使配置文件生效（实验：敲个beel然后按Tab键，如果补全了说明配置成功了）
$ echo $REDIS_HOME                                           # 查看是否能获取到环境变量的值
```

#### 九、启动各个节点上的Redis
```bash
$ redis-server /usr/redis-4.0.14/redis.conf                  # 根据指定配置文件启动Redis
$ tail -111f /usr/redis-4.0.14/log/redis-server.log          # 查看日志
```

#### 十、到集群任意一台机器上执行命令，创建集群(注意：集群最少三个主节点)
```bash
# --replicas 1 是指定副本数量，1就是指每一个主节点，都有一个从节点，现在共6台机器，那就是3主3备(主从分配规则: 按照创建命令的填写顺序，先创建主节点，再创建从节点)
$ redis-trib.rb create --replicas 1 server001:7000 server002:7001 server003:7002 server004:7003 server005:7004 server006:7005
```


#### 五、配置Redis开机启动
##### 5.1、复制Redis启动脚本到 /etc/init.d/redis
```bash
$ sudo cp /usr/redis-4.0.14/utils/redis_init_script /etc/init.d/redis
```

##### 5.2、修改Redis启动脚本[vi /etc/init.d/redis]
```bash
EXEC=/usr/redis-4.0.14/bin/redis-server                      # redis服务脚本所在目录
CLIEXEC=/usr/redis-4.0.14/bin/redis-cli                      # redis客户端脚本所在目录
CONF="/usr/redis-4.0.14/redis.conf"                          # redis配置文件在目录(注意：这个配置是带"双引号"的)
```

##### 5.3、redis启动和关闭(Redis设置密码之后，关闭服务的问题)
```bash
$ service redis start                                        # 启动redis
$ service redis stop                                         # 停止redis(注意：如果redis有设置密码，该命令无法停止redis)
$ redis-cli -p 6379 -a jiang shutdown                        # 停止redis(用登录密码连接redis然后执行shutdown停止redis)
$ sudo chkconfig redis on                                    # 设置redis开机启动
$ sudo chkconfig redis off                                   # 关闭redis开机启动
```

#### 六、删除redis安装包和解压目录(看情况而定)

