# Docker 笔记
## 一、基础
1，查看信息
docker info


2，查看运行状态
docker ps


3，在8080端口运行nginx

docker run -p 8080:80 -d nginx

通过docker ps查看实例id:b21852d129ab


4，停止容器
docker stop 5bc196f929a9


5，拷贝文件到docker

docker cp index.html b21852d129ab://usr/share/nginx/html/

停止后，docker将恢复初始

要保存：docker commit -m "add file" b21852d129ab

通过docker images查看，发现产生了匿名的image镜像

重新保存：docker commit -m "add file" b21852d129ab nginx-myinit

nginx-myinit        latest              8744ee81c7d1        3 seconds ago       109MB
<none>              <none>              87c68fe08474        2 minutes ago       109MB

删除匿名的image:

docker rmi 87c68fe08474

查看运行历史：docker ps -a

移除运行历史：docker rm 5fdcb33a432a b21852d129ab


6，简要介绍
docker pull 获取image

docker build 创建image

docker images 列出image

docker run 运行container

docker ps 列出container

docker rm 删除已结束的container

docker rmi 删除image

docker cp 在host和container之间拷贝文件

docker commit 保存改动为新的image


7，Dockerfile

./dockerdemo/Dockerfile

```
FROM alpine:latest
MAINTAINER libo
CMD echo "hello docker"
```

创建hello_docker镜像: docker build -t hello_docker dockerdemo/

hello_docker        latest              9c6607c526a9        15 minutes ago      5.58MB

运行：docker run hello_docker

8,第2个Dockerfile

./dockerdemo2/Dockerfile、index.html

```
FROM ubuntu
MAINTAINER libo
RUN apt-get update
RUN apt-get install -y nginx
COPY index.html /var/www/html
ENTRYPOINT ["/usr/sbin/nginx", "-g", "daemon off;"]
EXPOSE 80
```
docker build -t hello_docker2 dockerdemo2/

hello_docker2       latest              91204e1be09a        20 seconds ago      151MB

docker run -p 80:80 -d hello_docker2

9，Dockerfile语法

FROM 指定基础镜像

RUN 执行命令

ADD 添加文件(支持远程文件)

COPY 拷贝文件

CMD 执行命令

EXPOSE 暴露端口

WORKDIR 指定工作路径

MAINTAINER 维护者

ENV 设置环境变量

ENTRYPOINT 容器入口

USER 指定用户

VOLUME mount point


##  二、Volume

挂载bak目录到容器home：docker run -v /Users/libo/Desktop/bak/:/
home -p 80:80 -d hello_docker2

进入容器：docker exec -it agitated_hopper /bin/bash  
其中agitated_hopper为ps时看到的容器名

变量：$PWD/bak/ 当前目录的bak文件

docker create -v $PWD/data:/var/mydata --name data_container ubuntu

创建容器卷，将当前data目录挂在到容器中， 容器名为data_container，
基于ubuntu镜像

docker run --volume-from data_container hello_docker2

将容器卷挂在到某容器中

docker run --volumes-from data_container -d hello_docker2

docker exec -it epic_mayer /bin/bash 进入容器，在var/data目录可以看到文件

该容器卷，可以跨多个容器，被多个容器共享

## 三、镜像仓库registry
术语

host 宿主机

image 镜像

container 容器

registry 仓库

daemon 守护程序

client 客户端

搜索镜像

docker search 

拉取镜像到本地

docker pull nginx

docker push my/nginx

docker login登录，按提示输入用户名和密码

新建tag: docker tag hello_docker2 aibany/hello_docker2:0.0.1

推送到仓库：docker push aibany/hello_docker2:0.0.1

提交后可能有缓存，search不到，可以用拉取：docker pull aibany/hello_docker2:0.0.1


## 四、docker-compose
目录结构:

libodeMacBook-Pro:ghost libo$ tree
```

├── data
├── docker-compose.yml
├── ghost
│   ├── Dockerfile
│   └── config.js
└── nginx
    ├── Dockerfile
    └── nginx.conf
```
    
其中:docker-compose.yml:

```
version: '2'

networks:
  ghost:

services:
  ghost-app:
    build: ghost
    networks:
      - ghost
    depends_on:
      - db
    ports:
      - "2368:2368"

  nginx:
    build: nginx
    networks:
      - ghost
    ports:
      - "80:80"
    depends_on:
      - ghost-app
  db:
    image: "mysql:5.7.15"
    networks:
      - ghost
    environment:
      MYSQL_ROOT_PASSWORD: mysqlroot
      MYSQL_USER: ghost
      MYSQL_PASSWORD: ghost
    volumes:
      - $PWD/data:/var/libmysql
    ports:
    - "3306:3306"

```

ghost/Dockerfile

```
FROM ghost
COPY ./config.js /var/lib/ghost/config.js
EXPOSE 2368
#CMD ["npm", "start", "--production"]
```
ghost/config.js
```
var path = require('path'),
    config;

config = {
    production: {
        url: 'http://mytestblog.com',
        mail: {},
        database: {
            client: 'mysql',
            connection: {
                host: 'db',
                user: 'ghost',
                database: 'ghost',
                port: '3306',
                charset: 'utf-8'
            },
            debug: false
        },
        paths: {
            contentPath: path.join(process.env.GHOST_CONTENT,'/')
        },
        server: {
            host: '0.0.0.0',
            port: '2368'
        }
    }
};
module.exports =config;
```
nginx/Dockfile

```
FROM nginx
COPY nginx.conf /etc/nginx/nginx.conf
EXPOSE 80
```
nginx/nginx.conf
```
worker_processes auto;
events {
        worker_connections 1024;
}
http {
        server {
                listen 80;
                location / {
                        proxy_pass http://ghost-app:2638;
                }
        }
}
```

开始：docker-compose up -d

停止：docker-compose stop

移除：docker-compose rm

重新构建：docker-compose build

可能只能通过 localhost:2368访问







