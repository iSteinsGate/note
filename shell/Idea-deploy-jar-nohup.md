#  IDEA Alibaba Cloud 一键部署到远程服务器(nohup方式)

## 远程服务器

**账户**

```
IP:192.169.199.12
用户：root
密码：123456
```

**目录结构**

![image-20200910160403138](https://raw.githubusercontent.com/iSteinsGate/note/master/assets/image-20200910160403138.png)

- hycloud-admin.jar（可运行jar包）
- start.sh （启动脚本）
- stop.sh （停止脚本）
- viewlog.sh （查看控制台日志脚本）

**脚本**

- start.sh

  ```shell
  PID=`ps -ef | grep hycloud-admin.jar | grep -v grep | awk '{print $2}'`
  # -z判断空字符串
  if [ -z "$PID" ]
  then 
  nohup java -jar /usr/local/hycloud/hycloud-admin.jar --spring.profiles.active=prod >/usr/local/hycloud/catalina.txt 2>&1 &
  echo start application
  else 
      echo application is already start in pid:$PID
      echo kill $PID
      kill -9 $PID 
  nohup java -jar /usr/local/hycloud/hycloud-admin.jar --spring.profiles.active=prod >/usr/local/hycloud/catalina.txt 2>&1 &
  echo start application
  fi
  ```

- stop.sh

  ```shell
  PID=`ps -ef | grep hycloud-admin.jar | grep -v grep | awk '{print $2}'`
  if [ -z "$PID" ]
  then
      echo Application is already stopped
  else
      echo kill $PID
      kill -9 $PID
  fi
  ```

- viewlog.sh

  ```bash
  tail -f /usr/local/hycloud/catalina.txt
  ```

## IDEA配置

示意图：

![Udn5qK.png](https://s1.ax1x.com/2020/07/15/Udn5qK.png)

1. 使用Maven Build方式，选择服务器IP

2. 设置远程目录

```
/usr/local/hycloud
```

3. before launch设置

   添加maven goal命令

   ![UdesyV.png](https://s1.ax1x.com/2020/07/15/UdesyV.png)

   命令一：

   ![UdmEfs.png](https://s1.ax1x.com/2020/07/15/UdmEfs.png)

   命令二：

   ![image-20200715102146254](C:%5CUsers%5Ctsinghui%5CAppData%5CRoaming%5CTypora%5Ctypora-user-images%5Cimage-20200715102146254.png)

4. advanced高级设置

![Udm73n.png](https://s1.ax1x.com/2020/07/15/Udm73n.png)

Command含义：部署之后启动项目并且打开控制台日志

```
cd /usr/local/hycloud&&/usr/local/hycloud/start.sh&&/usr/local/hycloud/viewlog.sh
```



## 脚本合并(合并start.sh和stop.sh)

例如：hycloud-admin.sh

- ./hycloud-admin.sh start
- ./hycloud-admin.sh stop
- ./hycloud-admin.sh status

```bash
#!/bin/bash
app_name='hycloud-admin'
PID=`ps -ef | grep ${app_name}.jar | grep -v grep | awk '{print $2}'`
# $1表示输入的第一个参数：例如：./hycloud-admin.sh start $1表示start
if [ "$1" = "start" ];then
        # -z判断pid为空字符串
    if [ -z "$PID" ];then
    # 启动程序
    nohup java -jar /usr/local/hycloud/${app_name}.jar --spring.profiles.active=prod >/usr/local/hycloud/catalina.txt 2>&1 &
    echo ${app_name} start successfully
    echo "提示:使用 ./viewlog.sh 查看日志"
    else 
        echo ${app_name} is running in pid:$PID
        echo -e 提示：停止程序请使用以下命令 "\n./${app_name} stop"
    fi
elif [ "$1" = "stop" ];then
     if [ -z "$PID" ];then
        echo ${app_name} is not running
    else
        kill -9 $PID
        # 停止程序
        echo kill $PID
        echo ${app_name} is stopped
    fi
elif [ "$1" = "status" ];then
      if [ -z "$PID" ];then
 echo ${app_name} is stopped
      else
        echo ${app_name} is already running in pid:$PID
      fi
else
        echo -e 请输入下列命令:"\n./${app_name}.sh start" "./${app_name}.sh stop" "./${app_name} status"
fi
```

## 快速创建脚本文件

```bash
cat <<EOF >hycloud-admin.sh
#!/bin/bash
app_name='hycloud-admin'
PID=\`ps -ef | grep \${app_name}.jar | grep -v grep | awk '{print \$2}'\`
# \$1表示输入的第一个参数：例如：./hycloud-admin.sh start \$1表示start
if [ "\$1" = "start" ];then
        # -z判断pid为空字符串
    if [ -z "\$PID" ];then
    # 启动程序
    nohup java -jar /usr/local/hycloud/\${app_name}.jar --spring.profiles.active=prod >/usr/local/hycloud/catalina.txt 2>&1 &
    echo \${app_name} start successfully
    echo "提示:使用 ./viewlog.sh 查看日志"
    else
        echo \${app_name} is running in pid:\$PID
        echo -e 提示：停止程序请使用以下命令 "\n./\${app_name} stop"
    fi
elif [ "\$1" = "stop" ];then
     if [ -z "\$PID" ];then
        echo \${app_name} is not running
    else
        kill -9 \$PID
        # 停止程序
        echo kill \$PID
        echo \${app_name} is stopped
    fi
elif [ "\$1" = "status" ];then
      if [ -z "\$PID" ];then
 echo \${app_name} is stopped
      else
        echo \${app_name} is already running in pid:\$PID
      fi
else
        echo -e 请输入下列命令:"\n./\${app_name}.sh start" "./\${app_name}.sh stop" "./\${app_name} status"
fi
EOF
chmod +x ./hycloud-admin.sh
```

