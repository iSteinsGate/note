# mysql备份还原

mysqldump工具介绍

```bash
mysqldump--导出工具

#导出单个数据库：结构 无数据
[root@localhost ~]#mysqldump -h127.0.0.1 -uroot -p --opt --no-data db_name >~/db_name.sql

#导出单个数据库：有数据 无结构
[root@localhost ~]#mysqldump -h127.0.0.1 -uroot -p --opt --no-create-info db_name >~/db_name.sql

#导出单个数据库：结构+数据
[root@localhost ~]#mysqldump -h127.0.0.1 -uroot -p --opt db_name >~/db_name.sql

#导出单个数据库：结构+数据+函数+存储过程
[root@localhost ~]#mysqldump -h127.0.0.1 -uroot -p --opt -R db_name >~/db_name.sql

#导出多个数据库：结构
[root@localhost ~]#mysqldump -h127.0.0.1 -uroot -p --opt --databases db_name1 db_name2 >~/db_name.sql

#导出单个表：结构 无数据
[root@localhost ~]#mysqldump -h127.0.0.1 -uroot -p --opt --no-data -d db_name table>~/table_name.sql

#导出单个表：结构 包含数据
[root@localhost ~]#mysqldump -h127.0.0.1 -uroot -p --opt dbname  tablename>～/tablename.sql

mysql导入：
[root@localhost ~]#mysql -u root -p 
[root@localhost ~]#use  要导入的数据库
[root@localhost ~]#source /home/1.sql

或者
[root@localhost ~]#mysql -u root -p< /home/1.sql

注：
##--opt==--add-drop-table + --add-locks + --create-options + --disables-keys + --extended-insert + --lock-tables + --quick + --set+charset
##默认使用--opt，--skip-opt禁用--opt参数
```

## 备份

1. 直接使用命令

```bash
#只备份数据和结构
mysqldump -h127.0.0.1 -uroot -p123456 dbname  >/usr/local/db-backup/dbname-2020-0810.bak.sql
#备份数据和机构以及函数和存储过程
mysqldump -h127.0.0.1 -uroot -p123456 --opt -R dbname  >/usr/local/db-backup/dbname-2020-0810.bak.sql
```

2. 备份脚本（backup.sh）

```bash
#!/bin/bash
#保存备份数量，默认保存最近15天
number=15
#数据库ip
ip='127.0.0.1'
#数据库用户名
username='root'
#数据库密码
password='123456'
#备份的数据库名
databaseName='bdname'
#备份到目录,结尾不能加‘/’
backup_dir='/usr/local/db-backup'
#日期
d=`date +'%Y-%m%d-%H%M'`
if [ ! -d $backup_dir ]; 
then     
    mkdir -p $backup_dir; 
fi
echo "开始备份..."
echo "开始备份..." >> $backup_dir/log.txt
#备份命令
mysqldump -h${ip} -u${username} -p${password} --opt -R ${databaseName}  >${backup_dir}/${databaseName}-${d}.bak.sql
echo "正在检查是否需要删除多余备份....." >> $backup_dir/log.txt
#写创建备份日志
echo "create $backup_dir/$databaseName-$d.bak.sql" >> $backup_dir/log.txt
#找出需要删除的备份
delfile=`ls -l -crt  $backup_dir/*.sql | awk '{print $9 }' | head -1`
#判断现在的备份数量是否大于$number
count=`ls -l -crt  $backup_dir/*.sql | awk '{print $9 }' | wc -l`
if [ $count -gt $number ]
then
  #删除最早生成的备份，只保留number数量的备份
  rm $delfile
  #写删除文件日志
  echo "delete $delfile" >> $backup_dir/log.txt
  echo "删除多余的备份文件:$delfile"
fi
echo "备份成功" >> $backup_dir/log.txt
echo "备份完成"
```

## 还原

1. 直接运行命令

```
mysql -h${ip} -u${username} -p${password} ${databaseName} < ${bak_dir}
```

2. 脚本恢复（restore.sh）

```shell
#!/bin/bash
#还原到哪一个IP上的数据库
ip="127.0.0.1"
#数据库名称
username="root"
#数据库密码
password="123456"
#还原到哪一个数据库
databaseName="dbname"
#需要参数：已备份的sql文件
if [ -f "$1" ];
then
bak_dir="$1"
echo "开始还原数据库"
echo "执行命令：mysql -h${ip} -u${username} -p${password} ${databaseName} < ${bak_dir}"
mysql -h${ip} -u${username} -p${password} ${databaseName} < ${bak_dir}
echo "还原成功"
else echo "需要参数：备份的sql文件路径"
fi
```

使用

```bash
restore.sh /home/hycloud-user-mysql/back.bak.sql
```

## 快速创建备份还原脚本

```bash
cat <<EOF >backup.sh
#!/bin/bash
#保存备份数量，默认保存最近15天
number=15
#数据库ip
ip='127.0.0.1'
#数据库用户名
username='root'
#数据库密码
password='123456'
#备份的数据库名
databaseName='bdname'
#备份到目录,结尾不能加‘/’
backup_dir='/usr/local/db-backup'
#日期
d=\`date +'%Y-%m%d-%H%M'\`
if [ ! -d \$backup_dir ]; 
then     
    mkdir -p \$backup_dir; 
fi
echo "开始备份..."
echo "开始备份..." >> \$backup_dir/log.txt
#备份命令
echo "执行备份命令："
echo "mysqldump -h\${ip} -u\${username} -p\${password} --opt -R \${databaseName}  >\${backup_dir}/\${databaseName}-\${d}.bak.sql"
mysqldump -h\${ip} -u\${username} -p\${password} \${databaseName}  >\${backup_dir}/\${databaseName}-\${d}.bak.sql
echo "正在检查是否需要删除多余备份....." >> \$backup_dir/log.txt
#写创建备份日志
echo "create \$backup_dir/\$databaseName-\$d.bak.sql" >> \$backup_dir/log.txt
#找出需要删除的备份
delfile=\`ls -l -crt  \$backup_dir/*.sql | awk '{print $9 }' | head -1\`
#判断现在的备份数量是否大于$number
count=\`ls -l -crt  \$backup_dir/*.sql | awk '{print $9 }' | wc -l\`
if [ \$count -gt \$number ]
then
  #删除最早生成的备份，只保留number数量的备份
  rm \$delfile
  #写删除文件日志
  echo "delete \$delfile" >> \$backup_dir/log.txt
  echo "删除多余的备份文件:\$delfile"
fi
echo "备份成功" >> \$backup_dir/log.txt
echo "备份完成"
EOF
chmod +x ./backup.sh
cat <<EOF >restore.sh
#!/bin/bash
#还原到哪一个IP上的数据库
ip="127.0.0.1"
#数据库名称
username="root"
#数据库密码
password="123456"
#还原到哪一个数据库
databaseName="dbname"
#需要参数：已备份的sql文件
if [ -f "\$1" ];
then
bak_dir="\$1"
echo "开始还原数据库"
echo "执行命令：mysql -h\${ip} -u\${username} -p\${password} \${databaseName} < \${bak_dir}"
mysql -h\${ip} -u\${username} -p\${password} \${databaseName} < \${bak_dir}
echo "还原成功"
else echo "需要参数：备份的sql文件路径"
fi
EOF
chmod +x ./restore.sh
ls
```

执行成功后

![](https://raw.githubusercontent.com/iSteinsGate/picture/master/assets/image-20200910145353310.png)

## linux定时任务

1. cron语法

```bash
# cron语法
minute  hour  day-of-month month-of-year day-of-week      commands 
00-59  00-23    01-31         01-12       0-6(0 is sunday) 
#例子：
#1.每隔一个小时执行
0 */1 * * * /usr/local/db-backup/backup.sh
#2.每天晚上23点执行
0 23 * * * /usr/local/db-backup/backup.sh
```

2. cron文件（db-backup.cron）

   可以定时启动多个脚本

```
0 23 * * * /usr/local/db-backup/backup1.sh
0 22 * * * /usr/local/db-backup/backup2.sh
```

3. 启动定时任务

```bash
# 启用定时任务
crontab /usr/local/db-backup/db-backup.cron
# 查看当前用户的定时任务
crontab -l
#删除当前用户的定时任务
crontab -r
```

![](https://raw.githubusercontent.com/iSteinsGate/picture/master/assets/image-20200910153223670.png)

