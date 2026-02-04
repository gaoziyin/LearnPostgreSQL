# 2.1 psql 高级用法

## 📚 概述

本章深入探讨 psql 的高级功能，包括元命令的高级用法、变量系统、脚本编程、条件执行等专业技巧。

### 🎯 学习目标

- 掌握 psql 变量系统和条件执行
- 熟练使用脚本编程功能
- 了解高级输出控制和自动化技巧

---

## 🔧 变量系统

### psql 变量

psql 支持两种变量：内置变量和用户自定义变量。

```sql
-- 设置变量
\set myvar 'Hello World'
\set count 100

-- 使用变量 (注意冒号前缀)
SELECT :'myvar' AS message;
SELECT :count AS num;

-- 查看所有变量
\set

-- 取消变量
\unset myvar
```

### 内置变量

| 变量 | 说明 |
|------|------|
| `DBNAME` | 当前数据库名 |
| `USER` | 当前用户 |
| `HOST` | 主机地址 |
| `PORT` | 端口号 |
| `ENCODING` | 客户端编码 |
| `LASTOID` | 最后插入的 OID |
| `ERROR` | 最后的错误状态 |
| `ROW_COUNT` | 上次查询返回的行数 |

```sql
-- 使用内置变量
\echo 'Connected to' :DBNAME 'as' :USER
\echo 'Host:' :HOST ':' :PORT
```

### 查询结果存入变量

```sql
-- 使用 \gset 将查询结果存入变量
SELECT count(*) AS user_count FROM users \gset

-- 使用变量
\echo 'Total users:' :user_count

-- 带前缀存储多个值
SELECT min(id) AS min, max(id) AS max FROM users \gset stats_
\echo 'ID range:' :stats_min 'to' :stats_max
```

---

## 📜 脚本编程

### 条件执行

```sql
-- \if, \elif, \else, \endif 条件控制
\if :is_production
    \echo 'WARNING: Running in production!'
    \set safety_check true
\else
    \echo 'Development mode'
    \set safety_check false
\endif
```

### 循环与批处理

```bash
#!/bin/bash
# 批处理脚本示例: process_databases.sh

DATABASES=$(psql -U postgres -t -c "SELECT datname FROM pg_database WHERE datistemplate = false")

for db in $DATABASES; do
    echo "Processing database: $db"
    psql -U postgres -d $db -c "VACUUM ANALYZE;"
done
```

### 事务控制脚本

```sql
-- transaction_script.sql
\set ON_ERROR_STOP on
\set AUTOCOMMIT off

BEGIN;

-- 业务操作
INSERT INTO audit_log (action, created_at) VALUES ('batch_start', now());

UPDATE accounts SET balance = balance * 1.05 WHERE account_type = 'savings';

INSERT INTO audit_log (action, created_at) VALUES ('batch_end', now());

COMMIT;

\echo 'Transaction completed successfully'
```

---

## 📊 高级输出控制

### 格式化输出

```sql
-- 表格边框样式
\pset border 2
\pset linestyle unicode

-- 列对齐
\pset columns 120

-- 自定义分隔符
\pset fieldsep '|'

-- 记录分隔符
\pset recordsep '\n\n'

-- 表格标题
\pset title 'User Report - Generated at ' `date`
SELECT * FROM users LIMIT 5;
\pset title
```

### 输出到不同格式

```sql
-- HTML 输出
\pset format html
\o report.html
SELECT * FROM sales_summary;
\o
\pset format aligned

-- CSV 输出
\pset format csv
\o data.csv
SELECT * FROM users;
\o
\pset format aligned

-- LaTeX 输出
\pset format latex
\o table.tex
SELECT * FROM products LIMIT 10;
\o
```

### 管道输出

```sql
-- 输出到外部命令
\o | head -20
SELECT * FROM large_table;
\o

-- 输出到压缩文件
\o | gzip > backup.sql.gz
SELECT * FROM important_data;
\o
```

---

## 🔄 高级元命令

### 对象信息查询

```sql
-- 查看表的详细信息
\d+ users

-- 查看函数定义
\sf my_function

-- 查看视图定义
\sv my_view

-- 查看触发器
\dy

-- 查看规则
\dew

-- 查看扩展信息
\dx+ pg_stat_statements

-- 查看表空间
\db+

-- 查看访问权限
\dp users
\z users
```

### 系统目录查询

```sql
-- 自定义元命令 (使用 \gexec)
SELECT 'VACUUM ' || tablename || ';'
FROM pg_tables
WHERE schemaname = 'public'
\gexec

-- 生成并执行 DDL
SELECT 'CREATE INDEX idx_' || column_name || ' ON users(' || column_name || ');'
FROM information_schema.columns
WHERE table_name = 'users' AND column_name LIKE '%_id'
\gexec
```

---

## 📝 .psqlrc 配置

### 高级配置示例

```sql
-- ~/.psqlrc

-- 设置提示符
\set PROMPT1 '%[%033[1;32m%]%n@%/%R%#%[%033[0m%] '
\set PROMPT2 '%[%033[1;32m%]%n@%/%R%#%[%033[0m%] '

-- 历史设置
\set HISTSIZE 10000
\set HISTCONTROL ignoredups

-- 自动完成
\set COMP_KEYWORD_CASE upper

-- 显示设置
\timing on
\pset null '∅'
\pset linestyle unicode
\pset border 2

-- 有用的别名
\set version 'SELECT version();'
\set extensions 'SELECT * FROM pg_available_extensions ORDER BY name;'
\set activity 'SELECT pid, usename, datname, state, query FROM pg_stat_activity WHERE state != ''idle'' ORDER BY query_start DESC;'
\set locks 'SELECT * FROM pg_locks WHERE NOT granted;'
\set dbsize 'SELECT datname, pg_size_pretty(pg_database_size(datname)) FROM pg_database ORDER BY pg_database_size(datname) DESC;'
\set tablesize 'SELECT tablename, pg_size_pretty(pg_total_relation_size(schemaname || ''.'' || tablename)) FROM pg_tables WHERE schemaname = ''public'' ORDER BY pg_total_relation_size(schemaname || ''.'' || tablename) DESC LIMIT 20;'

-- 自动补全关键字大写
\set COMP_KEYWORD_CASE upper

-- 启动消息
\echo 'Welcome to PostgreSQL!'
\echo 'Custom commands: :version :extensions :activity :locks :dbsize :tablesize'
\echo ''
```

### 使用自定义别名

```sql
-- 执行预定义查询
:version
:activity
:dbsize
:tablesize
```

---

## 📊 流程图

```mermaid
flowchart TD
    subgraph Variables["变量系统"]
        SET[\set var value]
        USE[使用 :var]
        GSET[\gset 存储结果]
    end
    
    subgraph Control["流程控制"]
        IF[\if 条件判断]
        LOOP[Shell 循环]
        ERROR[ON_ERROR_STOP]
    end
    
    subgraph Output["输出控制"]
        FORMAT[\pset format]
        FILE[\o 文件输出]
        PIPE[\o | 管道]
    end
    
    subgraph Config["配置文件"]
        PSQLRC[.psqlrc]
        PROMPT[提示符]
        ALIAS[别名]
    end
    
    Variables --> Control
    Control --> Output
    Output --> Config
```

---

## 🎯 实战案例

### 案例 1: 自动化数据库报告

```sql
-- daily_report.sql
\timing off
\pset footer off
\pset tuples_only off

\echo '=========================================='
\echo ' PostgreSQL Daily Report'
\echo ' Generated:' `date`
\echo '=========================================='
\echo ''

\echo '1. Database Sizes'
\echo '-----------------'
SELECT datname AS "Database",
       pg_size_pretty(pg_database_size(datname)) AS "Size"
FROM pg_database
WHERE datistemplate = false
ORDER BY pg_database_size(datname) DESC;

\echo ''
\echo '2. Table Statistics (Top 10)'
\echo '----------------------------'
SELECT schemaname || '.' || relname AS "Table",
       n_live_tup AS "Live Rows",
       n_dead_tup AS "Dead Rows",
       last_vacuum AS "Last Vacuum",
       last_analyze AS "Last Analyze"
FROM pg_stat_user_tables
ORDER BY n_live_tup DESC
LIMIT 10;

\echo ''
\echo '3. Long Running Queries (>1min)'
\echo '-------------------------------'
SELECT pid,
       now() - query_start AS "Duration",
       usename AS "User",
       left(query, 60) AS "Query"
FROM pg_stat_activity
WHERE state = 'active'
  AND now() - query_start > interval '1 minute'
ORDER BY query_start;

\echo ''
\echo '4. Cache Hit Ratio'
\echo '------------------'
SELECT round(100.0 * sum(blks_hit) / nullif(sum(blks_hit) + sum(blks_read), 0), 2) AS "Cache Hit %"
FROM pg_stat_database;

\echo ''
\echo '=========================================='
\echo ' End of Report'
\echo '=========================================='
```

### 案例 2: 交互式维护脚本

```sql
-- maintenance.sql
\set ON_ERROR_STOP on

\echo 'PostgreSQL Maintenance Script'
\echo ''

-- 检查是否为超级用户
SELECT usesuper AS is_super FROM pg_user WHERE usename = current_user \gset

\if :is_super
    \echo 'Running as superuser - full maintenance available'
    
    \echo ''
    \echo 'Running VACUUM ANALYZE on all databases...'
    \! vacuumdb -U postgres --all --analyze --verbose
    
    \echo ''
    \echo 'Reindexing system catalogs...'
    REINDEX SYSTEM :DBNAME;
    
\else
    \echo 'Running as regular user - limited maintenance'
    
    \echo ''
    \echo 'Running VACUUM on current database...'
    VACUUM VERBOSE;
    
\endif

\echo ''
\echo 'Maintenance completed!'
```

---

## 💡 最佳实践

1. **使用 ON_ERROR_STOP**: 脚本中启用以便在错误时停止
2. **变量化敏感数据**: 避免在脚本中硬编码密码
3. **配置 .psqlrc**: 提高日常工作效率
4. **使用别名**: 为常用查询创建快捷方式
5. **格式化输出**: 根据需求选择合适的输出格式

---

## ❓ 常见问题

<details>
<summary><strong>Q: 如何在脚本中使用密码？</strong></summary>

使用 `.pgpass` 文件或环境变量 `PGPASSWORD`，避免在脚本中明文存储密码。
```bash
# 使用环境变量 (不推荐用于生产)
PGPASSWORD=mypass psql -U postgres -d mydb -f script.sql

# 推荐: 使用 .pgpass
# ~/.pgpass: localhost:5432:mydb:postgres:mypass
chmod 600 ~/.pgpass
```
</details>

<details>
<summary><strong>Q: 如何调试 psql 脚本？</strong></summary>

```sql
-- 显示执行的命令
\set ECHO all

-- 显示错误详情
\set VERBOSITY verbose

-- 在错误时停止
\set ON_ERROR_STOP on
```
</details>

---

[⬅️ 上一章: psql 入门](../../module-1-basics/1.3-psql-basics/README.md) | [返回目录](../../README.md) | [下一章: pgAdmin ➡️](../2.2-pgadmin/README.md)
