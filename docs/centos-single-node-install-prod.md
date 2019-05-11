#### 一、环境准备
```bash
$ sudo mkdir /usr/redis-4.0.14                                # 创建redis安装目录
$ sudo yum install -y gcc                                     # 安装gcc编译环境
$ wget http://download.redis.io/releases/redis-4.0.14.tar.gz  # 下载安装包
$ sudo tar -zxvf redis-4.0.14.tar.gz -C ./                    # 解压到当前目录
```

#### 二、编译安装以及测试是否安装成功
```bash
$ cd redis-4.0.14                                            # 跳转到redis解压目录下
$ sudo make & make PREFIX=/usr/redis-4.0.14 install          # 编译，安装到 /usr/redis-4.0.14 目录
$ sudo scp -r redis.conf utils /usr/redis-4.0.14             # 拷贝配置文件和工具脚本到 /usr/redis-4.0.14 目录
$ cd /usr/redis-4.0.14/bin                                   # 到redis安装目录(/usr/redis-4.0.14/bin)

$ ./redis-server                                             # 启动redis，测试是否安装成功
$ ps -aux | grep redis                                       # 查看redis进程
$ kill -9 7869                                               # 杀掉redis进程
```

#### 三、配置环境变量[vi ~/.bashrc]
```bash
export REDIS_HOME=/usr/redis-4.0.14
PATH=$PATH:$REDIS_HOME/bin                                   # linux以 : 号隔开，windows以 ; 号隔开

$ source ~/.bashrc                                           # （系统重读配置）在各个机器上执行使配置文件生效（实验：敲个beel然后按Tab键，如果补全了说明配置成功了）
$ echo $REDIS_HOME                                           # 查看是否能获取到环境变量的值
```

#### 四、修改[vi /usr/redis-4.0.14/redis.conf]配置文件(配置项都存在，只需要修改值即可)
```bash
bind 0.0.0.0                                                 # 绑定IP(允许那些IP可以访问，0.0.0.0是所有IP都可以访问)
port 6379                                                    # 端口
daemonize yes                                                # 以后台进程方式启动redis
requirepass jiang                                            # 配置登录密码是jiang
protected-mode yes                                           # 开启保护模式
```

#### 五、配置Redis开机启动
##### 5.1、复制Redis启动脚本到 /etc/init.d/redis
```bash
$ sudo cp /usr/redis-4.0.14/utils/redis_init_script /etc/init.d/redis
```

##### 5.2、修改Redis启动脚本[vi /etc/init.d/redis]
```bash
EXEC=/usr/redis-4.0.14/src/redis-server                      # redis服务脚本所在目录
CLIEXEC=/usr/redis-4.0.14/src/redis-cli                      # redis客户端脚本所在目录
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

