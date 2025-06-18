---
title: MySQL 8.4.5 在 Debian 上的安装配置指南
date: 2025-06-18 17:23:00 +0800
categories: [关系型数据库]
tags: [mysql]
pin: false
---

## 1. 前置准备

### 安装必要的依赖包

```bash
sudo apt update
# mysql 服务运行以及下载安装包需要
sudo apt install -y libaio1 libnuma1 libmecab2 libssl3 wget
# mysql cli工具运行需要
sudo apt install -y libncurses5 libncurses5-dev libncursesw5
```

### 创建 MySQL 用户和组

```bash
sudo groupadd mysql
sudo useradd -r -g mysql -s /bin/false mysql
```

## 2. 下载和解压

### 下载 MySQL 8.4.5 minimal 版本

```bash
cd /tmp
wget https://cdn.mysql.com//Downloads/MySQL-8.4/mysql-8.4.5-linux-glibc2.28-x86_64-minimal.tar.xz
```

### 解压到安装目录

```bash
sudo tar -xf mysql-8.4.5-linux-glibc2.28-x86_64-minimal.tar.xz -C /usr/local/
sudo mv /usr/local/mysql-8.4.5-linux-glibc2.28-x86_64-minimal /usr/local/mysql
sudo chown -R mysql:mysql /usr/local/mysql
```

## 3. 创建数据目录和配置

### 创建数据目录

```bash
sudo mkdir -p /var/lib/mysql
sudo chown mysql:mysql /var/lib/mysql
sudo chmod 750 /var/lib/mysql
```

### 创建日志目录

```bash
sudo mkdir -p /var/log/mysql
sudo chown mysql:mysql /var/log/mysql
```

### 创建配置文件

```bash
sudo tee /etc/my.cnf > /dev/null <<EOF
[mysqld]
# 基本设置
user = mysql
pid-file = /var/run/mysqld/mysqld.pid
socket = /var/run/mysqld/mysqld.sock
port = 3306
basedir = /usr/local/mysql
datadir = /var/lib/mysql

# 日志设置
log-error = /var/log/mysql/error.log
slow_query_log = 1
slow_query_log_file = /var/log/mysql/slow.log
long_query_time = 2

# 字符集设置
character-set-server = utf8mb4
collation-server = utf8mb4_unicode_ci

# InnoDB 设置
innodb_buffer_pool_size = 128M
innodb_redo_log_capacity = 100M
innodb_flush_log_at_trx_commit = 1
innodb_lock_wait_timeout = 50

# 网络设置
max_connections = 200
max_connect_errors = 10
table_open_cache = 64
max_allowed_packet = 4M
binlog_cache_size = 1M
max_heap_table_size = 8M
tmp_table_size = 16M

# MySQL 8.0+ 已移除查询缓存功能，无需配置

[mysql]
default-character-set = utf8mb4

[client]
port = 3306
socket = /var/run/mysqld/mysqld.sock
default-character-set = utf8mb4
EOF
```

## 4. 初始化 MySQL

### 创建 PID 文件目录

```bash
sudo mkdir -p /var/run/mysqld
sudo chown mysql:mysql /var/run/mysqld
```

### 初始化数据库

```bash
sudo /usr/local/mysql/bin/mysqld --initialize --user=mysql --basedir=/usr/local/mysql --datadir=/var/lib/mysql
```

**重要：** 初始化后会生成临时密码，请记录下来：

```bash
sudo grep 'temporary password' /var/log/mysql/error.log
```

## 5. 创建 systemd 服务文件

### 创建 systemd 服务单元

```bash
sudo tee /etc/systemd/system/mysql.service > /dev/null <<EOF
[Unit]
Description=MySQL Community Server
Documentation=man:mysqld(8)
Documentation=http://dev.mysql.com/doc/refman/en/using-systemd.html
After=network.target
After=syslog.target

[Install]
WantedBy=multi-user.target

[Service]
User=mysql
Group=mysql
Type=notify
TimeoutSec=0
PermissionsStartOnly=true
ExecStartPre=/bin/mkdir -p /var/run/mysqld
ExecStartPre=/bin/chown mysql:mysql /var/run/mysqld
ExecStart=/usr/local/mysql/bin/mysqld --defaults-file=/etc/my.cnf
ExecReload=/bin/kill -HUP $MAINPID
KillMode=process
Restart=on-failure
RestartPreventExitStatus=1
PrivateTmp=false
EOF
```

## 6. 启动和配置服务

### 重新加载 systemd 配置

```bash
sudo systemctl daemon-reload
```

### 启动 MySQL 服务

```bash
sudo systemctl start mysql
sudo systemctl enable mysql
```

### 检查服务状态

```bash
sudo systemctl status mysql
```

## 7. 安全配置

### 设置环境变量（可选）

```bash
echo 'export PATH=$PATH:/usr/local/mysql/bin' >> ~/.bashrc
source ~/.bashrc
```

### 运行安全脚本

```bash
sudo /usr/local/mysql/bin/mysql_secure_installation
```

按提示操作：

1. 输入初始化时生成的临时密码
2. 设置新的 root 密码
3. 移除匿名用户
4. 禁用远程 root 登录
5. 移除测试数据库
6. 重新加载权限表

## 8. 测试连接

### 使用新密码连接

```bash
/usr/local/mysql/bin/mysql -u root -p
```

### 测试 SQL 命令

```sql
SELECT version();
SHOW DATABASES;
EXIT;
```

## 9. 常用管理命令

### 服务管理

```bash
# 启动服务
sudo systemctl start mysql

# 停止服务
sudo systemctl stop mysql

# 重启服务
sudo systemctl restart mysql

# 查看状态
sudo systemctl status mysql

# 查看日志
sudo journalctl -u mysql -f

# 开机自启
sudo systemctl enable mysql

# 禁用开机自启
sudo systemctl disable mysql
```

### 日志查看

```bash
# 查看错误日志
sudo tail -f /var/log/mysql/error.log

# 查看慢查询日志
sudo tail -f /var/log/mysql/slow.log
```

## 10. 故障排除

### 常见问题

1. **权限问题**：确保所有 MySQL 相关目录的所有者是 mysql 用户
2. **端口占用**：检查 3306 端口是否被占用
3. **配置文件错误**：检查 `/etc/my.cnf` 语法是否正确
4. **内存不足**：适当调整 `innodb_buffer_pool_size` 等参数

### 检查端口

```bash
sudo netstat -tlnp | grep 3306
```

### 检查进程

```bash
ps aux | grep mysql
```

## 注意事项

1. **备份重要**：定期备份数据库
2. **防火墙设置**：如需远程连接，需要开放 3306 端口
3. **定期更新**：关注 MySQL 安全更新
4. **监控资源**：监控 CPU、内存、磁盘使用情况
5. **日志轮转**：配置日志轮转防止日志文件过大

安装完成后，MySQL 就会作为系统服务运行，可以通过 systemctl 进行管理。
