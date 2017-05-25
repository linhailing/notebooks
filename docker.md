# 记录docker的一些简单的使用

### 拉取镜像(拉取镜像经常会失败，尝试几次就好了)

```
docker pull ubuntu:14.04
```

### 设置以守护进程在后台运行

```
docker run -dit --name my-lnmp ubuntu:14.04
```

### 进入容器内部

```
docker exec -it my-lnmp bin/bash
```

### 退出

```
exit
```

### 设置容器开机启动项

```
# 在.bashrc写入开机启动项
vim ~/.bashrc
 
# 开机启动项
service php7.0-fpm start
service mysql start
service nginx start
#tail -f /var/log/nginx/error.log
```

### 将容器打包成镜像

```
# 退出 Docker
exit

# 查看当前容器对应CONTAINER ID
docker ps -a

# 将容器打包镜像
docker commit 2c10b080cf8c new-lnmp
```

### 运行最终镜像

```
# 查看当前容器对应CONTAINER ID
docker ps -a

# 将 my-lnmp 容器暂停（不暂停也行，但当前配置 运行不了两个容器，用的阿里云低配，1核1G）
docker stop my-lnmp

# 查看新的镜像/Pull的镜像
docker images
```


### 创建容器和共享本地目录运行

```
 # 使用刚打包的镜像，创建容器
 docker run -dit -p 80:80 -p 3306:3306 -v /var/www/:/apps/  --name nginx-mysql-php7-composer new-lnmp /bin/bash
 # -p 端口映射
 # -v 本地目录映射到容器内
```
