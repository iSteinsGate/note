# Alibaba Cloud Toolkit IDEA一键部署jar教程(docker方式)

使用alibaba toolkit一键打包springboot 可执行jar部署到远程服务器docker中

## 脚本环境准备

![aMv7fs.png](https://s1.ax1x.com/2020/07/31/aMv7fs.png)

- Dockerfile(构建docker镜像的配置)

  ```dockerfile
  # 指定java8环境
  FROM java:8
  # 将jar包加入到
  ADD hycloud-admin.jar /hycloud-admin.jar
  # 配置运行命令：激活cloud配置
  ENTRYPOINT ["java","-Dspring.profiles.active=cloud","-jar","hycloud-admin.jar"]
  # 暴露端口
  EXPOSE 8089
  # 维护者
  MAINTAINER tsinghui
  ```

- build-image.sh(构建镜像脚本)

  ```bash
  #!/bin/bash
  # 删除镜像
  docker rmi hycloud-user:1.0
  # 打包新的镜像
  # -t 表示指定镜像仓库名称/镜像名称:镜像标签 .表示使用当前目录下的Dockerfile
  docker build -t hycloud-user:1.0 .
  ```

- startup.sh(启动镜像生成容器脚本)

  ```bash
  # 启动docker容器
  # -p：端口映射 --name:容器名称 -v目录挂载 -d:后台运行
  docker run  -p 8089:8089 --name hycloud-user -v /home/hanyan/docker/hycloud-user:/home/hycloud -d hycloud-user:1.0
  # $?:上个命令执行成功返回0，失败返回1
  if [ $? -eq 0 ]
  then echo "hycloud-user容器启动成功"
  docker logs -f --tail=100 hycloud-user
  else echo "hycloud-user容器启动失败"
  fi
  ```

- stop.sh(停止容器进程，删除容器脚本)

  ```bash
  #!/bin/bash
  # 停止容器进程
  docker stop hycloud-user
  if [ $? -eq 0 ]
  then
   echo "停止hycloud-user成功"
  else echo "停止hycloud-user失败"
  fi
  # 删除容器
  docker rm hycloud-user
  if [ $? -eq 0 ]
  then echo "删除hycloud-user容器成功"
  else echo "删除hycloud-user容器失败"
  fi
  ```

- viewlog.sh(查看控制台实时打印)

  ```bash
  #!/bin/bash
  # -f:跟踪实时日志；-t:显示时间戳；--tail 从日志末尾显示多少行日志
  docker logs -f --tail=100 hycloud-user
  ```

## 快速创建脚本文件

```bash
cat <<EOF >Dockerfile
# 指定java8环境
FROM java:8
# 将jar包加入到
ADD hycloud-admin.jar /hycloud-admin.jar
# 配置运行命令：激活cloud配置
ENTRYPOINT ["java","-Dspring.profiles.active=cloud","-jar","hycloud-admin.jar"]
# 暴露端口
EXPOSE 8089
# 维护者
MAINTAINER tsinghui
EOF
;
cat <<EOF >build-image.sh
#!/bin/bash
# 删除镜像
docker rmi hycloud-user:1.0
# 打包新的镜像
# -t 表示指定镜像仓库名称/镜像名称:镜像标签 .表示使用当前目录下的Dockerfile
docker build -t hycloud-user:1.0 .
EOF
;
cat <<EOF >startup.sh
# 启动docker容器
# -p：端口映射 --name:容器名称 -v目录挂载 -d:后台运行
docker run  -p 8089:8089 --name hycloud-user -v /home/hanyan/docker/hycloud-user:/home/hycloud -d hycloud-user:1.0
# $?:上个命令执行成功返回0，失败返回1
if [ $? -eq 0 ]
then echo "hycloud-user容器启动成功"
else echo "hycloud-user容器启动失败"
fi
EOF
;
cat <<EOF >stop.sh
#!/bin/bash
# 停止容器进程
docker stop hycloud-user
if [ $? -eq 0 ]
then
 echo "停止hycloud-user成功"
else echo "停止hycloud-user失败"
fi
# 删除容器
docker rm hycloud-user
if [ $? -eq 0 ]
then echo "删除hycloud-user容器成功"
else echo "删除hycloud-user容器失败"
fi
EOF
;
cat <<EOF >viewlog.sh
#!/bin/bash
# -f:跟踪实时日志；-t:显示时间戳；--tail 从日志末尾显示多少行日志
docker logs -f --tail=100 hycloud-user
EOF
chmod +x ./build-image.sh ./startup.sh ./stop.sh ./viewlog.sh
```



## IDEA 配置

1. 配置服务器host

2. 配置target directory：服务器目标文件夹

3. 添加before launch

4. 配置上传成功后执行的命令

   切换目录 > 停止旧的程序 > 编译镜像 > 启动程序 > 查看日志
   
   ```bash
   cd /home/hanyan/docker/hycloud-user&&./stop.sh&&./build-image.sh&&./startup.sh&&./viewlog.sh
   ```

![alDJBD.png](https://s1.ax1x.com/2020/07/31/alDJBD.png)

上传成功后执行的命令：

![aWATYT.png](https://s1.ax1x.com/2020/08/07/aWATYT.png)