

查看mysql占用大小

```mysql
use information_schema;
# 查看整个库大小
select concat(round(sum(DATA_LENGTH/1024/1024),2),'MB') as data  from TABLES;
# 查看指定库
select concat(round(sum(DATA_LENGTH/1024/1024),2),'MB') as data  from TABLES where table_schema='databasename';
```

