---
title: 在Debian上使用安装PostgreSQL数据库
date: 2025-07-01 11:46:00 +0800
categories: [关系型数据库]
tags: [PostgreSQL]
pin: false
---

在 Debian 12 上安装 PostgreSQL 数据库，可以通过 `apt` 命令完成。下面是安装方法，并详细解释安装后涉及的各类文件的位置和作用，哪些是安装时生成的，哪些是运行时生成的。

## 一、安装 PostgreSQL

### 1.1 使用apt安装

```bash
sudo apt update
sudo apt install postgresql
```

这条命令会安装当前 Debian 12 默认的软件源中 PostgreSQL 的主版本（通常是 PostgreSQL 13、14 或 15，视当前源为准）。我在 2025.07.01 安装的时候，是15。

### 1.2 安装验证

APT 安装 PostgreSQL 后会自动做以下事情：

1. 创建一个系统用户 `postgres`。
2. 初始化数据目录 `/var/lib/postgresql/15/main`。
3. 启动 PostgreSQL 服务。
4. 创建默认数据库 `postgres` 和默认用户 `postgres`，这个用户是 postgrs 的超级用户。

可以使用以下命令查看一些与 postgres 相关的基本信息

```bash
# 查看 PostgreSQL 的版本
psql --version

# 查看 PostgreSQL 服务状态
sudo systemctl status postgresql

# 列出安装的 PostgreSQL 版本及配置路径
ls /etc/postgresql/
ls /etc/postgresql/15/main/

# 查看数据目录位置（可以在 postgresql.conf 中确认）
grep data_directory /etc/postgresql/15/main/postgresql.conf
```

你可以用下面命令以超级用户身份访问数据库：

```bash
sudo -u postgres psql
```

## 二、安装完成后文件的结构

按类型分类说明 PostgreSQL 安装后，可以关注一下涉及的部分文件及它们的路径：

### 2.1 🛠 可执行文件（命令行工具）

这些文件用于管理和使用 PostgreSQL：

|     命令     |          说明           |             路径             |
| ------------ | ----------------------- | ---------------------------- |
| `psql`       | PostgreSQL 的交互式终端 | `/usr/bin/psql`              |
| `createdb`   | 创建数据库              | `/usr/bin/createdb`          |
| `createuser` | 创建用户                | `/usr/bin/createuser`        |
| `pg_ctl`     | 控制 PostgreSQL 服务    | `/usr/lib/postgresql/15/bin/` |
| `postgres`   | 数据库主服务进程        | `/usr/lib/postgresql/15/bin/` |

这些 **可执行文件** 都是在安装时由 APT 安装产生的。

### 2.2 📦 依赖文件和库文件

系统运行 PostgreSQL 所需的依赖共享库：

|    类型    |                路径示例                |
| ---------- | -------------------------------------- |
| 动态库     | `/usr/lib/x86_64-linux-gnu/libpq.so.*` |
| 支持脚本等 | `/usr/share/postgresql/`               |

这些文件也都在安装过程中生成，用于支持 PostgreSQL 的运行环境。

### 2.3 ⚙️ 配置文件

配置文件用于控制数据库的行为。

|       文件        |      说明      |                   路径                    |
| ----------------- | -------------- | ----------------------------------------- |
| `postgresql.conf` | 主配置文件     | `/etc/postgresql/15/main/postgresql.conf` |
| `pg_hba.conf`     | 认证与访问控制 | `/etc/postgresql/15/main/pg_hba.conf`     |
| `pg_ident.conf`   | 身份映射配置   | `/etc/postgresql/15/main/pg_ident.conf`   |

这些文件在安装时会生成初始模板，你可以手动修改来调整数据库配置。

### 2.4 📁 数据库数据文件

这是 PostgreSQL 存储数据的地方（即你创建的数据库、表、索引等都在这里）：

|   内容   |              路径              |
| -------- | ------------------------------ |
| 数据文件 | `/var/lib/postgresql/15/main/` |
| WAL 日志 | 同上目录内的 `pg_wal` 子目录   |

⚠️ 这些是 **安装后初始化数据库时生成的**，是 **运行时持续写入和更新的内容**。

### 2.5 🪵 日志文件

默认 PostgreSQL 的日志写入到 systemd journal，但可以配置单独日志文件：

|     类型     |                 路径（默认）                 |              说明              |
| ------------ | -------------------------------------------- | ------------------------------ |
| Systemd 日志 | 使用 `journalctl -u postgresql` 查看         | 默认日志存于 systemd           |
| 文件日志     | `/var/log/postgresql/postgresql-15-main.log` | 如果配置了 `logging_collector` |

如果启用日志文件功能，日志是 **运行时生成和更新的**。

### 2.6 🧾 Systemd 服务配置

用于启动、停止 PostgreSQL 的服务脚本：

|      内容      |                     路径                     |            说明             |
| -------------- | -------------------------------------------- | --------------------------- |
| systemd unit   | `/lib/systemd/system/postgresql.service`     | 主服务管理单元文件          |
| 各实例配置文件 | `/etc/systemd/system/postgresql@.service.d/` | 可选的用户定制 service 配置 |

这些 systemd 配置文件是 **安装时生成的**，你可以用 `systemctl` 控制数据库。

## 三、账号授权示例

我们可以分别使用 PostgreSQL 提供的 `CREATE ROLE` 和权限控制系统 + `pg_hba.conf` 中的访问控制配置来实现访问权限的控制。

### 3.1 进入 PostgreSQL 管理终端

首先以超级用户身份进入 PostgreSQL：

```bash
sudo -u postgres psql
```

---

### 3.2 开发环境 - 创建开发账号（允许任意 IP，拥有创建数据库和所有权限）

#### ✅ 3.2.1 目标

* 用户名：`develop`
* 密码：如 `dev123`
* 允许从任意 IP 登录
* 可创建数据库、创建表、读写任意数据

#### 🔧 3.2.2 操作

```sql
-- 创建账号
CREATE ROLE develop WITH LOGIN PASSWORD 'dev123';

-- 允许创建数据库（开发用）
ALTER ROLE develop CREATEDB;

-- （可选）也允许创建角色（开发权限较大时）
-- ALTER ROLE develop CREATEROLE;

-- 授予对已有数据库的连接权限
GRANT CONNECT ON DATABASE postgres TO develop;
```

#### 🌍 3.2.3 配置 `pg_hba.conf` 允许任意 IP 登录

编辑文件（路径一般为 `/etc/postgresql/15/main/pg_hba.conf`）：

```bash
sudo vim /etc/postgresql/15/main/pg_hba.conf
```

添加一行：

```conf
host    all             develop         0.0.0.0/0               md5
```

> `0.0.0.0/0` 表示任意 IP，这在开发环境中可以接受，但生产中应避免。

#### 🌐 3.2.4 配置 PostgreSQL 监听地址

PostgreSQL 默认只监听 localhost，需要修改配置文件允许外部连接：

```bash
sudo vim /etc/postgresql/15/main/postgresql.conf
```

找到 listen_addresses 行，修改为：

```bash
# 原来可能是：
# listen_addresses = 'localhost'

# 修改为（开发环境）：
listen_addresses = '*'
```

> ⚠️ 安全提醒：listen_addresses = '*' 表示监听所有网络接口，这在开发环境可以接受，但生产环境应该指定具体的 IP 地址。

如果系统启用了防火墙，需要开放 PostgreSQL 端口（默认 5432）。

### 3.3 **开发环境 - 更安全的账号（限制特定网段）**

#### ✅ 3.3.1 目标

* 用户名：`devnet`
* 限制只能从 `192.168.0.0/16` 网段访问
* 其他权限与上面类似

#### 🔧 3.3.2 操作

```sql
CREATE ROLE devnet WITH LOGIN PASSWORD 'devnet123';
ALTER ROLE devnet CREATEDB;
GRANT CONNECT ON DATABASE postgres TO devnet;
```

编辑 `pg_hba.conf`：

```conf
host    all             devnet         192.168.0.0/16          md5
```

这么配置可实现只允许 `192.168.x.x` 的机器访问。

### 3.4 生产环境 - 创建应用账号（最小权限原则）

#### ✅ 3.4.1 目标

* 用户名：`app_user`
* 密码：`prodpass`
* 仅能访问某一个数据库（比如 `app_db`）
* 在该数据库中拥有查询、插入、更新、删除权限（不能建库、删库等）

#### 🔧 3.4.2 操作

```sql
-- 创建数据库
CREATE DATABASE app_db;

-- 创建账号
CREATE ROLE app_user WITH LOGIN PASSWORD 'prodpass';

-- 授予连接权限
GRANT CONNECT ON DATABASE app_db TO app_user;

-- 切换到数据库
\c app_db

-- 授权使用 public 模式下的表
GRANT USAGE ON SCHEMA public TO app_user;

-- 授权 DML 权限
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA public TO app_user;

-- 如果未来有新表，确保权限自动继承（非常关键）
ALTER DEFAULT PRIVILEGES IN SCHEMA public
GRANT SELECT, INSERT, UPDATE, DELETE ON TABLES TO app_user;
```

编辑 `pg_hba.conf` 限制访问来源：

```conf
host    app_db          app_user       10.0.0.0/24             md5
```

> `10.0.0.0/24` 是你生产网络段的 IP，仅允许该网段机器访问。

### 3.5 重启 PostgreSQL

```bash
sudo systemctl reload postgresql
```

或者重启整个服务（通常不需要）：

```bash
sudo systemctl restart postgresql
```

小结（对照表）

| 环境类型 |   用户名   |           权限           |       访问限制        |
| -------- | ---------- | ------------------------ | --------------------- |
| 开发环境 | `develop`  | 任意数据库，创建权限     | 任意 IP (`0.0.0.0/0`) |
| 安全开发 | `devnet`   | 同上                     | 内网 IP 段            |
| 生产环境 | `app_user` | 仅访问特定库的特定表权限 | 生产服务网段          |

### 3.6 连接测试

**重启服务后**，可以测试连接是否正常：

```bash
# 从本地测试
psql -h localhost -U develop -d postgres

# 从远程测试（替换为实际的服务器 IP）
psql -h 192.168.0.100 -U develop -d postgres
```

如果连接失败，检查：

-PostgreSQL 服务是否正常运行：sudo systemctl status postgresql
-postgresql.conf 中的 listen_addresses 是否正确配置
-pg_hba.conf 中的访问规则是否正确
-防火墙是否开放了 5432 端口
-网络连通性是否正常

> 💡 提示：配置修改后必须重启服务才能生效。
