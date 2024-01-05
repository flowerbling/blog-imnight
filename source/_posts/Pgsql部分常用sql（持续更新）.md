---
title: Pgsql部分常用sql（持续更新）
date: 2024-01-05 15:33:34
tags: [Pgsql]
categories:
    - 数据库
---
> 有一些Sql使用频率高 不想每次都敲完 写一些sql的模版来用

## Pgsql模版

### 建表
```sql
CREATE TABLE t_xxx (
    id bigserial NOT NULL,
    xxx int8 NOT NULL,
    xxx varchar NOT NULL,
    created_at timestamp without time zone NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at timestamp without time zone NOT NULL DEFAULT CURRENT_TIMESTAMP,
    deleted_at timestamp without time zone NULL,
    PRIMARY KEY (id)
);

CREATE TRIGGER tr_updated_at
    BEFORE UPDATE ON t_xxx
    FOR EACH ROW EXECUTE PROCEDURE on_update_current_timestamp();
```

### 加字段
```sql
ALTER TABLE t_xxx ADD COLUMN xxx jsonb NOT NULL DEFAULT '{}'::jsonb;
```

### 查锁解锁
```sql
SELECT * FROM pg_locks WHERE relation::regclass = 't_xxx'::regclass; -- 查锁
SELECT pg_terminate_backend(97811); -- 释放锁
```

### 新增约束
```sql
ALTER TABLE t_xxx ADD CONSTRAINT xxx UNIQUE (xxx);
```

### 改字段名
```sql
ALTER TABLE t_xxx RENAME COLUMN old_xxx TO new_xxx;
```

### 改字段类型
```sql
ALTER TABLE t_xxx ALTER COLUMN xxx TYPE varchar;
```

### 修改字段默认值
```sql
ALTER TABLE t_xxx ALTER COLUMN xxx SET DEFAULT '';
```
