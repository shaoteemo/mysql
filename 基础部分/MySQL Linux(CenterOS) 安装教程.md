# MySQL Linux(CenterOS) 安装教程

## 写在前面

**本教程仅仅讲诉压缩包的安装方式**


撰写人：ShaoTeemo


## 环境信息

|           环境            | 版本  |
| :-----------------------: | :---: |
|     CentOS（x86_64）      |  7.6  |
| MySQL Community（x86_64） | 8.0.3 |

## 下载MySQL

1. Download URL：https://dev.mysql.com/downloads/mysql/

2. 选择Linux-Generic。上传下载的MySQL至Linux服务器

3. 或者使用

    ```bash
    $> wget https://cdn.mysql.com//Downloads/MySQL-8.0/mysql-8.0.30-linux-glibc2.12-x86_64.tar
    ```

![](http://rep.shaoteemo.com/mysql%2Finstall%2Fmysql_install_1.png)

## 基础环境

### 1.校验并安装以下库

```bash
$> yum search libaio  # search for info
$> yum install libaio # install library
$> yum install ncurses-compat-libs
```

## 解压

推荐解压在：`/usr/local`

### 1.使用命令解压

```bash
$> tar -xvf mysql-8.0.30-linux-glibc2.12-x86_64.tar
```

### 2.继续解压此文件：mysql-8.0.30-linux-glibc2.12-x86_64.tar.xz

```bash
$> tar -xvf mysql-8.0.30-linux-glibc2.12-x86_64.tar.xz
```

> MySQL Server 8.0.12 中压缩算法由 Gzip 改为 XZ；并且通用二进制文件的扩展名从 .tar.gz 更改为 .tar.xz。

### 解压后的效果

![](http://rep.shaoteemo.com/mysql%2Finstall%2Fmysql_install_2.png)

### 目录结构

| 目录            | 目录内容                             |
| :-------------- | :----------------------------------- |
| `bin`           | mysqld服务器、客户端和实用程序       |
| `docs`          | 信息格式的 MySQL 手册                |
| `man`           | Unix 手册页                          |
| `include`       | 包含（头）文件                       |
| `lib`           | 运行库                               |
| `share`         | 用于数据库安装的错误消息、字典和 SQL |
| `support-files` | 其他支持文件                         |

## 创建用户及用户组

### 1.创建组

```bash
$> groupadd mysql
```

### 2.创建用户

```bash
$> useradd -r -g mysql -s /bin/false mysql
```

> 因为用户仅用于所有权目的，而不是登录目的，所以**useradd**命令使用 `-r`和`-s /bin/false`选项来创建对您的服务器主机没有登录权限的用户。**如果您的useradd**不支持 这些选项，请忽略它们。

## 创建目录符号链接及添加目录至PATH

```bash
$> ln -s <full-path-to-mysql-VERSION-OS> mysql
```

该`ln`命令创建一个指向安装目录的符号链接。这使您可以更轻松地将其称为`/usr/local/mysql`.

为了避免在使用 MySQL 时总是输入客户端程序的路径名，可以将`/usr/local/mysql/bin` 目录添加到`PATH`变量中：

```bash
$> export PATH=$PATH:/usr/local/mysql/bin
```

## 配置启动配置文件my.cnf

1.在没有任何选项文件的情况下，服务器将以其默认设置启动。要明确指定 MySQL 服务器在启动时应使用的选项，请将它们放在选项文件中，例如 `/etc/my.cnf`or `/etc/mysql/my.cnf`。

2.使用如下配置信息

```ini
[mysqld]
basedir=/usr/local/mysql
datadir=/usr/local/mysql/data
port=3306
character-set-server=utf8mb4
max_connections=200
character-set-server=utf8mb4
default_storage_engine=	InnoDB

# General and Slow logging.
log-output=FILE
general-log=0
general_log_file=/usr/local/mysql/log/logger.log
slow-query-log=1
slow_query_log_file=/usr/local/mysql/log/logger-slow.log"
long_query_time=10
# Error Logging.
log-error=/usr/local/mysql/log/logger.err
thread_cache_size=10
key_buffer_size=8M
innodb_buffer_pool_size=128M
# Linux Constant
lower_case_table_names=1

[mysql]
default-character-set=utf8mb4
```


## 初始化数据目录

1.进入mysql根目录

```bash
$> cd /usr/local/mysql
```

2.新建数据文件夹并将目录用户和组所有权授予 `mysql`用户和`mysql` 组，并适当设置目录权限

```bash
$> mkdir data
$> chown mysql:mysql data
$> chmod 750 data
```

3.使用服务器初始化数据目录，包括`mysql`包含初始 MySQL 授权表的模式，这些授权表确定用户如何被允许连接到服务器

```bash
$> bin/mysqld --initialize --console --user=mysql
```

![](http://rep.shaoteemo.com/mysql%2Finstall%2Fmysql_install_3.png)

**上图最后一条日志的密码用于登录并连接MySQL**

> 如果在控制台中没有信息输出请移步至：`log/logger.err`中查看


## mysql启动

### 1.直接启动

```bash
$> /usr/local/mysql/bin/mysqld --user=mysql
```

**进入MySQL后需要变更密码才可以使用**

```sql
ALTER USER 'root'@'localhost' IDENTIFIED BY '设置的密码';
```

**变更MySQL访问IP为全局**

```sql
RENAME USER 'root'@'localhost' TO 'root'@'%';
```


### 2.配置自动启动脚本使用systemd（CentOS >= 7）

#### 2.1创建[mysql.service]服务

```bash
$> touch /usr/lib/systemd/system/mysql.service
# 注意systemctl 中规定、服务的配置文件要以.service 为后缀
```

复制如下内容

```properties
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


PIDFile=/usr/local/mysql/data/mysqld.pid

# Disable service start and stop timeout logic of systemd for mysqld service.
TimeoutSec=0

# Execute pre and post scripts as root
PermissionsStartOnly=true
# Needed to create system tables
#ExecStartPre=/usr/bin/mysqld_pre_systemd

# Start main service
ExecStart=/usr/local/mysql/bin/mysqld --daemonize --pid-file=/usr/local/mysql/data/mysqld.pid
#注意这里要加上 --daemonize 
# Use this to switch malloc implementation
# EnvironmentFile=-/etc/sysconfig/mysql

# Sets open_files_limit
LimitNOFILE = 5000
Restart=on-failure
RestartPreventExitStatus=1
PrivateTmp=false
```

使用systemctl进行配置

```bash
$> systemctl enable mysql
$> systemctl daemon-reload
```

一些常用的命令

```bash
# 启动服务
$> systemctl start 服务名
# 停止服务
$> systemctl stop 服务名
# 查看服务状态
$> systemctl status 服务名
# 查看服务列表
$> systemctl status -l
# 重启服务
$> systemctl restart 服务名
# 设置服务开机启动
$> systemctl enable 服务名
# 禁止服务开机启动
$> systemctl disable 服务名
# 查看所有已经开启的服务
$> systemctl list-units --type=service
# 重新加载配置文件
$> systemctl daemon-reload
```

## 参考文献

官方文档：https://dev.mysql.com/doc/refman/8.0/en/binary-installation.html

自启动部分：https://blog.csdn.net/weixin_53690696/article/details/124788290
