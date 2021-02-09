# Mysql锁

数据库遵循的是两段锁协议，将事务分成两个阶段，加锁阶段和解锁阶段（所以叫两段锁）

- 加锁阶段：在该阶段可以进行加锁操作。在对任何数据进行读操作之前要申请并获得S锁（共享锁，其它事务可以继续加共享锁，但不能加排它锁），在进行写操作之前要申请并获得X锁（排它锁，其它事务不能再获得任何锁）。加锁不成功，则事务进入等待状态，直到加锁成功才继续执行。
- 解锁阶段：当事务释放了一个封锁以后，事务进入解锁阶段，在该阶段只能进行解锁操作不能再进行加锁操作。

## 乐观锁

数据库使用版本（version)记录机制实现。每次读取时，将版本一起读出来，更新时带上版本标记，更新时需要对比读取来的版本与当前数据库的版本是否一致，相等则予以更新，每次更新都将该版本+1。

**举例：**

1、数据库表三个字段，分别是id、value、version

```sql
select id,value,version from TABLE where id = #{id}
```

2、每次更新表中的value字段时，为了防止发生冲突，需要这样操作

```sql
update TABLE
set value=2,version=version+1
where id=#{id} and version=#{version}
```

## 悲观锁

### 共享锁(S),读锁

解释：当事务T对数据A加上**共享锁**后,则其他事务只能对A再加**共享锁**,不能加**排他锁**,获得**共享锁**的事务只能读取数据,不能修改数据

加锁方式：

select ... lock in share mode

#### 情况一

事务T

```sql
BEGIN;
select * from test where id = 1 lock in share mode;
update test set value = 2 where id = 1;
```

其他事务

```sql
BEGIN; 
select * from test where id = 1 lock in share mode; -- 直接阻塞，等待事务T提交
```

#### 死锁

两个事务对数据A同时加共享锁后，对数据进行修改，可能会导致死锁

事务一

```sql
BEGIN;
select * from test where id = 1 lock in share mode; 
```

事务二

```sql
BEGIN; 
select * from test where id = 1 lock in share mode;
```

当事务一事务二都获取了共享锁，分别执行更新时会导致死锁

![sqRVW8.png](https://s3.ax1x.com/2021/01/25/sqRVW8.png)



### 排它锁（X锁）,又称写锁

select   ... **for update** 是一种**行级锁**,又叫**排他锁**

解释:当事务T对某数据A加上了**排他锁**,其余事务不能对A加任何锁,只能进行读操作,不能进行修改写操作.

#### 情况一

事务T对A加排他锁，其他事务读取可以

事务T

```sql
BEGIN;
select * from test where id = 1 for update;
```

其他事务

```sql
BEGIN
select * from test where id = 1 ; -- 可以直接读取
```

#### 情况二

事务T对A加排他锁，其他事务对其进行修改会阻塞，只有当事务T提交后，其他事务才能正常提交

事务T

```sql
BEGIN;
select * from test where id = 1 for update;
```

其他事务

```sql
BEGIN;
update test set id = 1 ; -- 会直接阻塞
```

#### 情况三

当事务T对数据A加排他锁后，其他事务加任何锁都会直接阻塞

事务T

```sql
BEGIN
select * from test where id = 1 for update;
```

其他事务

```sql
BEGIN;
SELECT * FROM test WHERE id = 1 lock in share mode -- 会直接阻塞
SELECT * FROM test WHERE id = 1 for update  -- 会直接阻塞
```

### 行锁

解释：当多个事务同时修改同一行数据时，后面的事务会处于阻塞状态。

对于update,delete,insert语句会自动给涉及的数据加锁。可能会出现死锁。

事务T

```
BEGIN;
update test set value = 2 where id = 1; -- 加行锁
```

其他事务

```
BEGIN;
update test set value = 2 where id = 1; -- 等待事务T提交后，才能执行
```

**注意：行级锁都是基于索引的，如果一条SQL语句用不到索引是不会使用行级锁的，会使用表级锁。**

### 间隙锁(next-key锁)



### 解除死锁

第一种

1. 查询是否锁表 `show OPEN TABLES where In_use > 0;`

2. 查询进程（如果您有SUPER权限，您可以看到所有线程。否则，您只能看到您自己的线程）
    `show processlist`

3. 杀死进程id（就是上面命令的id列）
    `kill id`

第二种

1. 查看当前的事务
    `SELECT * FROM INFORMATION_SCHEMA.INNODB_TRX;`

2. 查看当前锁定的事务
    `SELECT * FROM INFORMATION_SCHEMA.INNODB_LOCKS;`

3. 查看当前等锁的事务
    `SELECT * FROM INFORMATION_SCHEMA.INNODB_LOCK_WAITS;`
4. 杀死进程
    `kill 进程ID`




[美团技术团队-mysql锁](https://tech.meituan.com/2014/08/20/innodb-lock.html)