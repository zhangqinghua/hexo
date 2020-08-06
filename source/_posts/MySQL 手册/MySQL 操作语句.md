---
title: MySQL 操作语句

categories:
- MySQL 手册

date: 2020-07-07 00:00:09
---

select 

## 常见问题
1. ERROR 1292 (22007): Truncated incorrect DOUBLE value
update 操作时不能用 `and`， 应该用 `,`。   

```sql
update biz_warehouse_back set a = 1 and b = 2 where is_deleted = 'n'
```