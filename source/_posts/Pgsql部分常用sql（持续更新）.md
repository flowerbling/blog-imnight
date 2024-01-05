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
CREATE TABLE xxx (
    id bigserial NOT NULL,
    xxx int8 NOT NULL,
    xxx varchar NOT NULL,
    created_at timestamp without time zone NOT NULL DEFAULT CURRENT_TIMESTAMP,
    updated_at timestamp without time zone NOT NULL DEFAULT CURRENT_TIMESTAMP,
    deleted_at timestamp without time zone NULL,
    PRIMARY KEY (id)
);

CREATE TRIGGER tr_updated_at
    BEFORE UPDATE ON xxx
    FOR EACH ROW EXECUTE PROCEDURE on_update_current_timestamp();
```

### 加字段
```sql
ALTER TABLE xxx ADD COLUMN xxx jsonb NOT NULL DEFAULT '{}'::jsonb;
```

### 查锁解锁
```sql
SELECT * FROM pg_locks WHERE relation::regclass = 'chatbot_material_sources'::regclass; -- 查锁
SELECT pg_terminate_backend(97811); -- 释放锁
```

### 新增约束
```sql
ALTER TABLE xxx ADD CONSTRAINT xxx UNIQUE (xxx);
```
