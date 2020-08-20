# LearnDocker

## 1. 安装Docker

- 安装

```sh
# 下载关于Docker的依赖环境
yum install -y yum-utils device-mapper-persistent-data lvm2
# 设置Docker镜像源
yum-config-manager --add-repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
# 安装Docker
yum makecache fast
yum -y install docker-ce

# 启动
systemctl start docker
# 设置开机自启动
systemctl enable docker
# 测试
docker run hello-world
```

- 镜像网站

```
hub.docker.com
c.163yun.com/hub#/home
hub.daocloud.io
```

- 搭建私有仓库

```json
# /etc/docker/daemon.json
{
    "registry-mirrors":["https://registry.docker-cn.com"],
    "insecure-registries":["ip:port"]
}
# 重启服务
systemctl daemon-reload
systemctl restart docker
```

## 2. 基本操作

### 2.1 镜像的操作

```sh
# 拉去镜像到本地
docker pull 镜像名称[:tag]
# 举个例子
docker pull daocloud.io/library/tomcat:8.5.15-jre8
```

```sh
# 查看全部镜像
docker images
```

```sh
# 删除本地镜像
docker rmi 镜像的唯一标识
```

```sh
# 镜像的导入导出
# 导出
docker save -o 导出的路径 镜像id
# 导入
docker load -i 镜像文件
# 改名
docker tag 镜像id 镜像名称:版本
```

### 2.2容器的操作

```sh
# 运行容器
# 简单操作
docker run 镜像标识|镜像名称[:tag]
# 常用参数
docker run -d -p 宿主机端口:容器端口 --name 容器名称 镜像标识|镜像名称[:tag]
```

```sh
# 查看正在运行的容器
docker ps [-aq]
# -a: 查看全部容器
# -q: 只查看容器标识
```

```sh
# 查看容器日志
docker logs -f 容器id
# -f: 可以滚动查看日志的最后几行
```

```sh
# 进入容器内部
docker exec -it 容器id bash
```

```sh
# 停止容器
docker stop 容器id
docker stop $(docker ps -qa)
# 删除容器
docker rm 容器id
docker rm $(docker ps -qa)
```

```sh
# 启动容器
docker start 容器id
```

```sh
# 传输文件
docker cp 文件名称 容器id:容器内部路径
```

### 2.3数据卷

数据卷: 将宿主机的一个目录映射到容器的一个目录中

```sh
# 创建数据卷
docker volume create 数据卷名称
# 创建数据卷之后，默认会存放在/var/lib/docker/volumes/数据卷名称/_data
```

```sh
# 查看数据卷的详细信息
docker volume inspect 数据卷名称
```

```sh
# 查看全部数据卷
docker volume ls
```

```sh
# 删除数据卷
docker volume rm 数据卷名称
```

```sh
# 应用数据卷
# 将容器中自带的文件存储在存放路径中
docker run -v 数据卷名称:容器内部路径 镜像id
# 不会将容器中自带的文件存储在存放路径中
docker run -v 路径:容器内部路径 镜像id
```

## 3. 自定义镜像

```sh
# 1.创建一个Dockerfile文件，并且制定自定义镜像信息
# Dockerfile文件中常用的内容
from: 制定当前自定义镜像依赖的环境
copy: 将相对路径下的内容复制到自定义镜像中
workdir: 声明镜像的默认工作目录
cmd: 需要执行的命令(在workdir中执行的，cmd可以写多个，只以最后一个为准)
# 举个栗子
from daocloud.io/library/tomcat:8.5.15-jre8
copy ssm.war /usr/local/tomcat/webapps

# 2. 制作镜像
docker build -t 镜像名称:[tag] 文件路径
```

## 4. Docker-Compose

### 4.1 下载Docker-compose

- github下载docker-compose文件
- 赋予docker-compose文件执行权限
- 可以写到环境变量

### 4.2 管理容器

```
yml文件以key: value方式指定配置信息
多个配置信息以换行+缩进的方式区分
在docker-compose.yml文件中，不要使用制表符
```

```yml
version: '3.1'
services:
  mysql:                                        # 服务名称
    restart: always                             # 只要docker启动，容器也会一起启动
    image: daocloud.io/library/mysql:5.7.4      # 指定镜像镜像
    container_name: mysql                       # 指定容器名称
    ports:
      - 3306:3306                               # 指定端口号的映射
    environment:
      MYSQL_ROOT_PASSWORD: root                 # 指定MySQL的ROOT用户登录密码
      TZ: Asia/Shanghai                         # 指定时区
    volumes:
      - /opt/docker_mysql_tomcat/mysql_data:/var/lib/mysql  # 映射数据卷
  tomcat:
    restart: always                             # 只要docker启动，容器也会一起启动
    image: daocloud.io/library/tomcat:8.5.15-jre8      # 指定镜像镜像
    container_name: tomcat                       # 指定容器名称
    ports:
      - 8080:8080
    environment:
      TZ: Asia/Shanghai
    volumes:
      - /opt/docker_mysql_tomcat/tomcat_webapps:/usr/local/tomcat/webapps
      - /opt/docker_mysql_tomcat/tomcat_logs:/usr/local/tomcat/logs  
  
```

### 4.3 使用docker-compose命令管理容器

默认会在当前目录下找docker-compose.yml

```sh
# 启动管理的容器
docker-compose up -d
```

```sh
# 关闭并删除容器
docker-compose down
```

```sh
# 开启|关闭|重启已经存在的容器
docker-compose start|stop|restart
```

```sh
# 查看管理的容器
docker-compose ps
```

```sh
# 查看日志
docker-compose logs -f
```

### 4.4 docker-compose配置Dockerfile使用

```yml
version: '3.1'
services:
  ssm:
    restart: always
    build:                         # 构建自定义镜像          
      context: ../                 # 指定dockerfile文件所在路径
      dockerfile: Dockerfile       # 指定Dockerfile文件名称
    image: ssm:1.0.1
    container_name: ssm
    ports:
      - 8081:8080
    environment:
      TZ: Asia/Shanghai
```

```sh
# 可以直接启动给予docker-compose.yml以及Dockerfile文件够贱的自定义镜像
docker-compose up -d
# 如果自定义镜像不存在，会帮助我们构建出自定义镜像，如果自定义镜像已经存在，会直接运行这个自定义镜像
# 重新构建自定义镜像
docker-compose build
# 运行前，重新构建
docker-compose up -d --build
```

## 5. Docker CI/CD

