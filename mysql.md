## mysql逻辑结构

![](https://raw.githubusercontent.com/iSteinsGate/picture/master/images/20200920120926.png)

## 查看mysql库大小，表大小，索引大小

**说明：**

通过MySQL的 information_schema 数据库，可查询数据库中每个表占用的空间、表记录的行数；该库中有一个 TABLES 表，这个表主要字段分别是：

TABLE_SCHEMA : 数据库名
TABLE_NAME：表名
ENGINE：所使用的存储引擎
TABLES_ROWS：记录数
DATA_LENGTH：数据大小
INDEX_LENGTH：索引大小

```mysql
#使用MySQL的 information_schema 数据库
use information_schema;
```

### 库大小

```mysql
# 查看整个库大小
select concat(round(sum(DATA_LENGTH/1024/1024),2),'MB') as data  from TABLES;
# 查看指定库databasename
select concat(round(sum(DATA_LENGTH/1024/1024),2),'MB') as data  from TABLES where table_schema='databasename';
```

### 表大小

```mysql
#查看指定tablename表大小
select concat(round(sum(DATA_LENGTH/1024/1024),2),'MB') as data  from TABLES where table_schema='databasename' and table_name='tablename';

# 查看databasename数据库表占用明细
SELECT CONCAT(table_schema,'.',table_name) AS 'Table Name', CONCAT(ROUND(table_rows/1000000,2),'M') AS 'Number of Rows', CONCAT(ROUND(data_length/(1024*1024),2),'M') AS 'Data Size', CONCAT(ROUND(index_length/(1024*1024),2),'M') AS 'Index Size', CONCAT(ROUND((data_length+index_length)/(1024*1024),2),'M') AS'Total'FROM information_schema.TABLES WHERE table_schema LIKE 'databasename';
```

### 索引大小

```mysql
#查看指定库索引大小
SELECT CONCAT(ROUND(SUM(index_length)/(1024*1024), 2), ' MB') AS 'Total Index Size' FROM TABLES  WHERE table_schema = 'databasename';
#查看指定库指定表索引大小
SELECT CONCAT(ROUND(SUM(index_length)/(1024*1024), 2), ' MB') AS 'Total Index Size' FROM TABLES  WHERE table_schema = 'databasename' and table_name='tablename';
```

