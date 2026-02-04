# 1.3 psql 命令行工具入门

## 📚 概述

`psql` 是 PostgreSQL 的官方交互式命令行工具，是每位 DBA 必须掌握的核心工具。它提供了强大的数据库管理、查询执行和脚本功能。

### 🎯 学习目标

- 掌握 psql 的连接方式和基本操作
- 熟悉常用的元命令 (meta-commands)
- 了解 psql 的输出格式化选项

---

## 🔧 连接数据库

### 连接语法

```bash
psql [OPTIONS] [DBNAME [USERNAME]]
```

### 常用连接方式

```mermaid
flowchart LR
    subgraph Connection["连接方式"]
        LOCAL[本地连接]
        TCP[TCP/IP 连接]
        URI[URI 连接]
    end
    
    LOCAL --> |"psql -U postgres"| DB[(PostgreSQL)]
    TCP --> |"psql -h host -p 5432 -U user -d dbname"| DB
    URI --> |"psql postgresql://user:pass@host/db"| DB
```

### 连接示例

```bash
# 本地连接 (使用当前系统用户)
psql

# 指定用户和数据库
psql -U postgres -d mydb

# 完整 TCP 连接
psql -h 192.168.1.100 -p 5432 -U admin -d production

# 使用 URI 格式
psql "postgresql://admin:password@localhost:5432/mydb"

# 使用环境变量
export PGHOST=localhost
export PGPORT=5432
export PGUSER=postgres
export PGDATABASE=mydb
psql
```

### 连接参数

| 参数 | 长格式 | 说明 |
|------|--------|------|
| `-h` | `--host` | 数据库服务器地址 |
| `-p` | `--port` | 端口号 (默认 5432) |
| `-U` | `--username` | 用户名 |
| `-d` | `--dbname` | 数据库名 |
| `-W` | `--password` | 强制提示密码 |
| `-w` | `--no-password` | 不提示密码 |

---

## 📜 基础元命令

psql 的元命令以反斜杠 `\` 开头，用于执行管理操作。

### 帮助与信息

```sql
-- 获取帮助
\?              -- 显示所有元命令帮助
\h              -- SQL 命令帮助
\h SELECT       -- 特定命令帮助

-- 版本信息
SELECT version();
```

### 导航命令

```sql
-- 列出数据库
\l              -- 列出所有数据库
\l+             -- 详细信息 (含大小)

-- 切换数据库
\c mydb         -- 连接到 mydb
\c mydb admin   -- 以 admin 用户连接

-- 列出对象
\dt             -- 列出表
\dt+            -- 表详细信息
\di             -- 列出索引
\dv             -- 列出视图
\df             -- 列出函数
\dn             -- 列出模式
\du             -- 列出用户/角色
```

### 元命令速查表

```mermaid
graph TB
    subgraph List["列表命令 (\\d)"]
        DT["\dt - 表"]
        DI["\di - 索引"]
        DV["\dv - 视图"]
        DF["\df - 函数"]
        DN["\dn - 模式"]
        DU["\du - 角色"]
    end
    
    subgraph Desc["描述命令"]
        D["\d table - 表结构"]
        SF["\sf func - 函数定义"]
        SV["\sv view - 视图定义"]
    end
    
    subgraph IO["输入输出"]
        I["\i file - 执行文件"]
        O["\o file - 输出到文件"]
        COPY["\copy - 数据导入导出"]
    end
    
    subgraph Control["控制命令"]
        Q["\q - 退出"]
        TIMING["\timing - 计时"]
        X["\x - 扩展显示"]
    end
```

---

## 🎨 输出格式化

### 显示模式切换

```sql
-- 切换扩展显示 (垂直输出)
\x
\x auto         -- 自动选择

-- 查看表结构示例
\d users

-- 扩展模式下的输出更清晰
\x on
SELECT * FROM pg_stat_activity LIMIT 1;
```

### 输出格式

```sql
-- 设置输出格式
\pset format aligned    -- 对齐格式 (默认)
\pset format unaligned  -- 非对齐
\pset format html       -- HTML 格式
\pset format csv        -- CSV 格式

-- 设置边框
\pset border 0          -- 无边框
\pset border 1          -- 单边框
\pset border 2          -- 双边框

-- 设置空值显示
\pset null '[NULL]'
```

### 实用设置

```sql
-- 显示查询执行时间
\timing on

-- 显示执行的查询
\echo 'Running query...'

-- 变量设置
\set HISTSIZE 1000      -- 历史记录条数
\set COMP_KEYWORD_CASE upper  -- 关键字大写补全
```

---

## 📂 脚本执行

### 执行 SQL 文件

```bash
# 方法1: 启动时执行
psql -U postgres -d mydb -f script.sql

# 方法2: 交互模式执行
\i /path/to/script.sql

# 方法3: 管道执行
cat script.sql | psql -U postgres -d mydb
```

### 输出重定向

```sql
-- 输出到文件
\o /tmp/output.txt
SELECT * FROM users;
\o                      -- 关闭重定向

-- 静默模式 (只显示结果)
\t on
SELECT table_name FROM information_schema.tables;
\t off
```

---

## 📊 常用查询示例

### 数据库信息

```sql
-- 当前连接信息
\conninfo

-- 数据库大小
SELECT pg_database.datname,
       pg_size_pretty(pg_database_size(pg_database.datname)) as size
FROM pg_database
ORDER BY pg_database_size(pg_database.datname) DESC;

-- 表大小
SELECT tablename,
       pg_size_pretty(pg_total_relation_size(schemaname || '.' || tablename)) as size
FROM pg_tables
WHERE schemaname = 'public'
ORDER BY pg_total_relation_size(schemaname || '.' || tablename) DESC
LIMIT 10;
```

### 连接与进程

```sql
-- 当前连接
SELECT pid, usename, datname, client_addr, state, query
FROM pg_stat_activity
WHERE state != 'idle';

-- 终止连接
SELECT pg_terminate_backend(pid)
FROM pg_stat_activity
WHERE datname = 'mydb' AND pid <> pg_backend_pid();
```

### 锁信息

```sql
-- 查看锁
SELECT l.locktype, l.relation::regclass, l.mode, l.granted, a.usename, a.query
FROM pg_locks l
JOIN pg_stat_activity a ON l.pid = a.pid
WHERE l.relation IS NOT NULL;
```

---

## 🎯 实战案例

### 案例 1: 快速数据库健康检查

创建一个健康检查脚本 `health_check.sql`:

```sql
-- health_check.sql
\echo '=== PostgreSQL 健康检查 ==='
\echo ''

\echo '1. 版本信息'
SELECT version();

\echo ''
\echo '2. 数据库列表'
\l+

\echo ''
\echo '3. 活动连接'
SELECT datname, usename, state, count(*)
FROM pg_stat_activity
GROUP BY datname, usename, state
ORDER BY count(*) DESC;

\echo ''
\echo '4. 数据库大小 Top 5'
SELECT datname, pg_size_pretty(pg_database_size(datname)) as size
FROM pg_database
ORDER BY pg_database_size(datname) DESC
LIMIT 5;

\echo ''
\echo '5. 缓存命中率'
SELECT 
    sum(blks_hit) * 100.0 / nullif(sum(blks_hit) + sum(blks_read), 0) as cache_hit_ratio
FROM pg_stat_database;

\echo ''
\echo '=== 检查完成 ==='
```

执行脚本:

```bash
psql -U postgres -f health_check.sql
```

### 案例 2: 交互式表探索

```sql
-- 连接到数据库
\c mydb

-- 查看所有表
\dt

-- 查看特定表结构
\d users

-- 查看表索引
\di users*

-- 查看表数据 (带分页)
\pset pager on
SELECT * FROM users LIMIT 100;

-- 导出数据
\copy users TO '/tmp/users.csv' WITH CSV HEADER
```

---

## 📊 工作流程图

```mermaid
flowchart TD
    A[启动 psql] --> B{连接成功?}
    B -->|否| C[检查连接参数]
    C --> A
    B -->|是| D[进入交互模式]
    
    D --> E{操作类型}
    E -->|查询| F[执行 SQL]
    E -->|管理| G[使用元命令]
    E -->|脚本| H[执行文件 \i]
    
    F --> I[查看结果]
    G --> I
    H --> I
    
    I --> J{继续?}
    J -->|是| E
    J -->|否| K[\q 退出]
```

---

## 💡 最佳实践

1. **使用 .pgpass 文件**: 安全存储密码，避免命令行暴露
   ```
   # ~/.pgpass 格式: hostname:port:database:username:password
   localhost:5432:*:postgres:mypassword
   ```

2. **配置 .psqlrc**: 自定义 psql 启动设置
   ```sql
   -- ~/.psqlrc
   \timing on
   \pset null '[NULL]'
   \set HISTSIZE 2000
   \set COMP_KEYWORD_CASE upper
   ```

3. **使用扩展显示**: 宽表数据使用 `\x auto`

4. **记录历史**: psql 自动保存命令历史到 `~/.psql_history`

5. **使用 Tab 补全**: 表名、列名支持 Tab 键自动补全

---

## ❓ 常见问题

<details>
<summary><strong>Q: 如何清屏？</strong></summary>

```sql
-- Linux/Mac
\! clear

-- Windows
\! cls

-- 或使用快捷键 Ctrl+L
```
</details>

<details>
<summary><strong>Q: 如何查看上一条命令？</strong></summary>

使用方向键 ↑ 可以浏览命令历史。也可以使用 `\s` 显示历史记录。
</details>

<details>
<summary><strong>Q: 如何中断长时间运行的查询？</strong></summary>

按 `Ctrl+C` 可以中断当前执行的查询。
</details>

<details>
<summary><strong>Q: 如何编辑复杂查询？</strong></summary>

```sql
-- 打开外部编辑器编辑
\e

-- 编辑上一条查询
\e buffer

-- 设置编辑器
\setenv EDITOR vim
```
</details>

---

## 📚 延伸阅读

- [psql 官方文档](https://www.postgresql.org/docs/18/app-psql.html)
- [psql 技巧合集](https://wiki.postgresql.org/wiki/Psql_Tips)

---

[⬅️ 上一章: 安装与配置](../1.2-installation/README.md) | [返回目录](../../README.md) | [下一章: psql 高级用法 ➡️](../../module-2-core-tools/2.1-psql-advanced/README.md)
