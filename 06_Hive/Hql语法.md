# Hive 常用语法

## 1. Hive 时间常用语法

Hive 获取当前时间

```sql
select now();
select current_timestamp();
```

Hive 获取年月日

```sql
select yaer(now());
select month(now());
select day(now());
```

yyyy-MM-dd HH:mm:ss 标准时间日期格式

```sql
select date_format(now(), 'yyyy');
select data_format(now(), 'MM');
select data_format(now(), 'yyyy-MM-dd');
```

