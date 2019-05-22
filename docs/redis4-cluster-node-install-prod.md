#### 一、环境准备
```bash
$ sudo mkdir -p /usr/redis-4.0.14                            # 创建redis安装目录
$ wget -P /usr/redis-4.0.14 http://rubygems.org/downloads/redis-3.3.0.gem
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
logfile "/usr/redis-4.0.14/log/redis-server.log"             # 日志所在目录(注意：手动创建目录)
pidfile /usr/redis-4.0.14/redis.pid

cluster-enabled yes                                          # 打开集群模式
masterauth jiang                                             # 从节点同步数据时，访问主节点所使用的密码(Redis节点的密码)
cluster-config-file /usr/redis-4.0.14/cluster-nodes.conf     # 集群信息文件(这个文件是Redis集群自动生成的，如果要重新创建集群需删除各个节点的这个文件(删除此文件，将丢失集群slot(槽位)信息，请慎重))

# 以下配置一般不配
cluster-node-timeout 10000                                   # 集群节点超时时间
cluster-slave-validity-factor 5                              # 切换为主机的时间(0-10，值越小说明检查越频繁)
repl-ping-slave-period 5                                     # 检查主节点健康状态                        
repl-timeout 30                                              # 主从复制超时时间，该值要大于 repl-ping-slave-period 的值
```

#### 四、修改Redis启动脚本[vi /usr/redis-4.0.14/utils/redis_init_script]
```bash
REDISPORT=6379                                               # redis端口（需和redis配置文件里的一致）
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

#### 六、安装Rubby环境(集群所有节点都要安装，集群控制工具依赖环境)
```bash
$ yum install ruby rubygems
$ gem install --local /usr/redis-4.0.14/redis-3.3.0.gem
```

#### 七、修改集群控制工具的配置文件[vi /usr/local/share/gems/gems/redis-3.3.0/lib/redis/client.rb]为其指定Redis密码，集群所有节点都要修改(如果不知道client.rb文件在哪里，可使用该命令查找：find / -name 'client.rb')
```bash
:password => "jiang"
```

#### 八、配置Redis开机启动(集群所有节点都要配置)
##### 8.1、复制Redis启动脚本到 /etc/init.d/redis
```bash
$ sudo cp /usr/redis-4.0.14/utils/redis_init_script /etc/init.d/redis
```

##### 8.2、启动集群所有节点的Redis
```bash
$ service redis start                                        # 启动redis
$ service redis stop                                         # 停止redis(注意：如果redis有设置密码，该命令无法停止redis)
$ sudo chkconfig redis on                                    # 设置redis开机启动
$ sudo chkconfig redis off                                   # 关闭redis开机启动
```

#### 九、任选一台机器，创建集群(注意：集群最少三个主节点，创建节点时要使用IP否则可能会报：ERR Invalid node address specified 错误)
```bash
$ cd /usr/redis-4.0.14/bin
# --replicas 1 是指定副本数量，1就是指每一个主节点，都有一个从节点，现在共6台机器，那就是3主3备(主从分配规则: 按照创建命令的填写顺序，先创建主节点，再创建从节点)
$ ./redis-trib.rb create --replicas 1 192.168.229.133:6379 192.168.229.129:6379 192.168.229.134:6379 192.168.229.132:6379 192.168.229.137:6379 192.168.229.138:6379
```

#### 十、连接测试集群
```bash
$ cd /usr/redis-4.0.14/bin
$ ./redis-cli -c -p 6379 -a jiang                            # -c表示集群模式连接
$ cluster info                                               # 查看集群信息
$ cluster nodes                                              # 查看集群所有节点信息

$ set k1 k1
$ get k1 k1
```

#### 十一、集群新增主节点，任选一台已在集群以内的机器执行如下命令
```bash
$ ./redis-trib.rb                                                         # 查看 redis-trib.rb 脚本使用帮助  
$ ./redis-trib.rb add-node 192.168.83.143:6379 192.168.83.143:6379        # 集群添加一个主节点(第一个为新增节点，第二个为集群中已知存在节点(任意一个))
$ ./redis-trib.rb reshard 192.168.83.143:6379                             # 为新增的主节点分配slot(槽位)，不然它不能存储数据

How many slots do you want to move (from 1 to 16384)? 500                 # 你希望将多少个slot(槽位)移动到新的节点上，可以自己设置，比如500个slot(槽位)      
What is the receiving node ID? 2f9a662051adec43b8c8f567340cede230f4d25a   # 接收这些slot(槽位)的节点的id(输入新节点ID即可，ID可进入Redis客户端使用 cluster nodes命令查看)
Please enter all the source node IDs.
  Type 'all' to use all the nodes as source nodes for the hash slots.
  Type 'done' once you entered all the source nodes IDs.
Source node #1:all                                                        # 抽取那些节点的slot(槽位)到新节点中：all为所有主节点，可以直接填写ID，最后填写done表示确定）
```

#### 十二、集群删除主节点，任选一台已在集群以内的机器执行如下命令，但不要选要删除的那台节点(原理：先将slot(槽位)重新移动到其他主节点上去，再进行删除操作，不然存放的数据就丢失了)
```bash
$ ./redis-trib.rb reshard 192.168.83.143:6379                             # 移动slot(槽位)

M: 2f9a662051adec43b8c8f567340cede230f4d25a 192.168.83.143:6379
   slots:22-165,5461-5627,10923-11088 (477 slots) master
   0 additional replica(s)
[OK] All nodes agree about slots configuration.
>>> Check for open slots...
>>> Check slots coverage...
[OK] All 16384 slots covered.
How many slots do you want to move (from 1 to 16384)? 477                 # 这里填的就是192.168.83.143拥有的slot(槽位)数，上面有打印出来的
What is the receiving node ID? 94378bbe72297a47a86e4621c64cd73951f08b0b   # 这里是接收slot(槽位)的主节点id，我填的是192.168.229.132的id
Please enter all the source node IDs.
  Type 'all' to use all the nodes as source nodes for the hash slots.
  Type 'done' once you entered all the source nodes IDs.
Source node #1:2f9a662051adec43b8c8f567340cede230f4d25a                   # 这里是数据源节点，当然填的是192.168.83.143的Id了
Source node #2:done                                                       # 确认

# 删除节点
$ ./redis-trib.rb del-node 192.168.83.143:6379 2f9a662051adec43b8c8f5673  # 参数是IP和端口，以及节点的ID(ID可进入Redis客户端使用 cluster nodes命令查看)
```

#### 十五、集群新增从节点
```bash
$ ./redis-trib.rb add-node 192.168.83.143:6379 192.168.83.143:6379        # 集群添加一个从节点(第一个为新增节点，第二个为集群中已知存在节点(任意一个))
$ /usr/redis-4.0.15/bin/redis-cli -c -h 192.168.83.143 -p 6379 -a jiang   # 连接到刚刚新增的节点
$ cluster replicate 94378bbe72297a47a86e4621c64cd73951f08b0b              # 指定当前节点的主节点ID
```

#### 十六、集群删除从节点
```bash
$ ./redis-trib.rb del-node 192.168.83.143:6379 34f30d8c7e0ed7970b9be7714  # 参数是IP和端口，以及节点的ID(ID可进入Redis客户端使用 cluster nodes命令查看)
```


#### 十七、删除redis安装包和解压目录(看情况而定)

