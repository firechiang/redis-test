#### 一、环境准备
```bash
$ sudo mkdir /usr/redis-5.0.5                                # 创建redis安装目录
$ sudo yum install -y gcc                                    # 安装gcc编译环境
$ wget http://download.redis.io/releases/redis-5.0.5.tar.gz  # 下载安装包
$ sudo tar -zxvf redis-5.0.5.tar.gz -C ./                    # 解压到当前目录
```

#### 二、编译安装以及测试是否安装成功
```bash
$ cd redis-5.0.5                                             # 跳转到redis解压目录下
$ sudo make & make PREFIX=/usr/redis-5.0.5 install           # 编译，安装到 /usr/redis-5.0.5 目录
$ sudo scp -r redis.conf utils /usr/redis-5.0.5              # 拷贝配置文件和工具脚本到 /usr/redis-5.0.5 目录
$ cd /usr/redis-5.0.5/bin                                    # 到redis安装目录(/usr/redis-5.0.5/bin)

$ ./redis-server                                             # 启动redis，测试是否安装成功
$ ps -aux | grep redis                                       # 查看redis进程
$ redis-cli shutdown                                         # 停止redis
```

#### 三、修改[vi /usr/redis-5.0.5/redis.conf]配置文件(配置项都存在，只需要修改值即可)
```bash
bind 0.0.0.0                                                 # 绑定IP(允许那些IP可以访问，0.0.0.0是所有IP都可以访问)
port 6379                                                    # 端口
daemonize yes                                                # 以后台进程方式启动redis
requirepass jiang                                            # 配置登录密码是jiang
protected-mode yes                                           # 开启保护模式
dir /usr/redis-5.0.5/data                                    # 持久化数据所在目录(注意：手动创建目录)
logfile "/usr/redis-5.0.5/log/redis-server.log"              # 日志所在目录(注意：手动创建目录)
pidfile /usr/redis-5.0.5/redis.pid
```

#### 四、配置Redis开机启动
##### 4.1、复制Redis启动脚本到 /etc/init.d/redis
```bash
$ sudo cp /usr/redis-5.0.5/utils/redis_init_script /etc/init.d/redis
```

##### 4.2、修改Redis启动脚本[vi /etc/init.d/redis]
```bash
REDISPORT=6379                                               # redis端口（需和redis配置文件里的一致）
EXEC=/usr/redis-5.0.5/bin/redis-server                      # redis服务脚本所在目录
CLIEXEC=/usr/redis-5.0.5/bin/redis-cli                      # redis客户端脚本所在目录
CONF="/usr/redis-5.0.5/redis.conf"                          # redis配置文件在目录(注意：这个配置是带"双引号"的)

$CLIEXEC -p $REDISPORT -a jiang shutdown                     # 关闭redis时所使用的代码，加上 -a jiang(就是Redis密码)
```

##### 4.3、redis启动和关闭(Redis设置密码之后，关闭服务的问题)
```bash
$ service redis start                                        # 启动redis
$ service redis stop                                         # 停止redis(注意：如果redis有设置密码，该命令无法停止redis)
$ sudo chkconfig redis on                                    # 设置redis开机启动
$ sudo chkconfig redis off                                   # 关闭redis开机启动
```

#### 五、删除redis安装包和解压目录(看情况而定)

