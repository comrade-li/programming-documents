# MySQL相关文档

## 1.MySQL安装

以MySQL8.0为例。

### 1.1 解压二进制包安装

```shell
# 注意！！！Debian系Linux如果缺少libaio, 在遇到可在 https://pkgs.org/download/libaio 下载安装。

# 添加mysql用户组和用户
sudo groupadd mysql
sudo useradd -r -g mysql -s /bin/false mysql

# 解压MySQL二进制包到/opt并链接到/usr/local/mysql下
sudo tar -xvf mysql-8.0.37-linux-glibc2.28-x86_64.tar.xz -C /opt
sudo ln -s /opt/mysql-8.0.37-linux-glibc2.28-x86_64 /usr/local/mysql

# 创建mysql-files目录和data目录并授权
sudo mkdir /usr/local/mysql/mysql-files /usr/local/mysql/data 
sudo chown mysql:mysql /usr/local/mysql/mysql-files /usr/local/mysql/data
sudo chmod 750 /usr/local/mysql/mysql-files /usr/local/mysql/data

# 新建MySQL配置文件my.cnf,内容如下
sudo vim /etc/my.cnf

# 初始化数据文件夹
cd /usr/local/mysql
sudo ./bin/mysqld --defaults-file=/etc/my.cnf --initialize-insecure

# 添加systemd并启动MySQL服务
sudo vim /usr/lib/systemd/system/mysqld.service #内容如下
sudo systemctl start mysqld

# 将mysql客户端链接到/usr/bin目录下
sudo ln -s /usr/local/mysql/bin/mysql /usr/bin/mysql

# 登录MySQL服务并修改root密码
mysql -u root --skip-password
ALTER USER 'root'@'localhost' IDENTIFIED BY 'root-password';
```

MySQL配置文件/etc/my.cnf

```shell
[mysqld]
datadir=/usr/local/mysql/data
socket=/tmp/mysql.sock
port=3306
log-error=/usr/local/mysql/data/localhost.localdomain.err
user=mysql
secure_file_priv=/usr/local/mysql/mysql-files
local_infile=OFF
```

MySQL systemd配置文件如下

```shell
[Unit]
Description=MySQL Server
Documentation=man:mysqld(8)
Documentation=http://dev.mysql.com/doc/refman/en/using-systemd.html
After=network.target
After=syslog.target

[Install]
WantedBy=multi-user.target

[Service]
User=mysql
Group=mysql

# Have mysqld write its state to the systemd notify socket
Type=notify

# Disable service start and stop timeout logic of systemd for mysqld service.
TimeoutSec=0

# Start main service
# $MYSQL_OPTS暂时用不上
# ExecStart=/usr/local/mysql/bin/mysqld --defaults-file=/etc/my.cnf $MYSQLD_OPTS 
ExecStart=/usr/local/mysql/bin/mysqld --defaults-file=/etc/my.cnf

# Use this to switch malloc implementation
EnvironmentFile=-/etc/sysconfig/mysql

# Sets open_files_limit
LimitNOFILE = 10000

Restart=on-failure

RestartPreventExitStatus=1

# Set environment variable MYSQLD_PARENT_PID. This is required for restart.
Environment=MYSQLD_PARENT_PID=1

PrivateTmp=false
```

### 1.2 利用docker安装

```shell
# 拉取对应的镜像
docker pull mysql:8.0.37

# 创建宿主机映射到容器文件夹和配置文件
#    宿主机                 容器
# mysql/data    -->    /var/lib/mysql
# mysql/my.cnf  -->    /etc/my.cnf
cd ~/Data
mkdir -p mysql/data mysql/log
vim mysql/my.cnf # 文件内容如下

# 启动docker容器
docker run -p 3306:3306 --name mysql -v /home/lijing/Data/mysql/data:/var/lib/mysql -v /home/lijing/Data/mysql/my.cnf:/etc/my.cnf -e MYSQL_ROOT_PASSWORD=123456 -d mysql:8.0.37
```

MySQL配置文件my.cnf

```shell
[mysqld]
skip-host-cache
skip-name-resolve
datadir=/var/lib/mysql
socket=/var/run/mysqld/mysqld.sock
secure-file-priv=/var/lib/mysql-files
user=mysql
pid-file=/var/run/mysqld/mysqld.pid

[client]
socket=/var/run/mysqld/mysqld.sock
!includedir /etc/mysql/conf.d/
```
