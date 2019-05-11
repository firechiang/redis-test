#### 一、环境准备
```bash
$ sudo mkdir /usr/redis                                       # 创建redis安装目录
$ sudo yum install -y gcc                                     # 安装gcc编译环境
$ wget http://download.redis.io/releases/redis-4.0.14.tar.gz  # 下载安装包
$ sudo tar -zxvf redis-4.0.14.tar.gz -C /usr/redis            # 解压到 /usr/redis 目录
```

#### 二、编译安装以及测试是否安装成功
```bash
$ cd /usr/redis/redis-4.0.14                                  # 跳转到redis解压目录下
$ sudo make & make install                                    # 编译安装

$ ./src/redis-server                                          # 启动redis，测试是否安装成功
$ ps -aux | grep redis                                        # 查看redis进程
$ kill -9 7869                                                # 杀掉redis进程
```

#### 三、修改[vi /usr/redis/redis-4.0.14/redis.conf]配置文件(配置项都存在，只需要修改值即可)
```bash
bind 0.0.0.0                                                  # 绑定IP(允许那些IP可以访问，0.0.0.0是所有IP都可以访问)
port 6379                                                     # 端口
daemonize yes                                                 # 以后台进程方式启动redis
requirepass jiang                                             # 配置登录密码是jiang
protected-mode yes                                            # 开启保护模式
```

#### 四、配置Redis开机启动
##### 4.1、复制Redis启动脚本到 /etc/init.d/redis
```bash
$ sudo cp /usr/redis/redis-4.0.14/utils/redis_init_script /etc/init.d/redis
```

##### 4.2、修改Redis启动脚本[vi /etc/init.d/redis]
```bash
EXEC=/usr/redis/redis-4.0.14/src/redis-server                # redis服务脚本所在目录
CLIEXEC=/usr/redis/redis-4.0.14/src/redis-cli                # redis客户端脚本所在目录
CONF="/usr/redis/redis-4.0.14/redis.conf"                    # redis配置文件在目录(注意：这个配置是带"双引号"的)
```

##### 4.3、redis启动和关闭(Redis设置密码之后，关闭服务的问题)
```bash
$ service redis start                                        # 启动redis
$ service redis stop                                         # 停止redis(注意：如果redis有设置密码，该命令无法停止redis)
$ redis-cli -p 6379 -a jiang shutdown                        # 停止redis(用登录密码连接redis然后执行shutdown停止redis)
$ sudo chkconfig redis on                                    # 设置redis开机启动
$ sudo chkconfig redis off                                   # 关闭redis开机启动
```


