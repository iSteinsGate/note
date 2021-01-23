# Mysql锁



### 共享锁(S),读锁

select ... lock in share mode

```sql
事务一:
BEGIN   第一步
select * from test where id = 1 lock in share mode; 第二步
update test set value = 2 where id = 1; 第三步
commit 4
事务二:
BEGIN 第一步
select * from test where id = 1 lock in share mode; 第二步
update test set value = 3 where id = 1; 第三步
commit 4
# 当事务一事务二都执行完第二步;然后事务一执行更新和事务二执行更新后会导致思索
# 当事务一执行完第三步,事务二执行第二步会阻塞等待事务一提交后才能加锁
```



### 更新(U)

### 排他锁(X),行级锁

select   ... **for update** 是一种**行级锁**,又叫**排他锁**

```sql
事务一:
BEGIN
select * from test where id = 1 for update;
commit

# 当事务一开启事务执行 for update后,事务一会对id=1的数据行加排他锁,其他事务只能查询,但无法修改id=1的数据,只有当事务一提交后才能修改.
# 如果id=1已有排他锁,无法重复加锁.及当事务一未提交时,执行 select * from test where id = 1 for update会阻塞等待.其他id行则不会select * from test where id = 1 for update
```

