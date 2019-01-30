## 列出镜像

### 列出镜像

```bash
sudo docker images
```

### 拉取ubuntu镜像

```bash
sudo docker pull ubuntu:12.04
```

### 运行一个带标签的docker镜像

```bash
sudo docker run -it --name dalong_container ubuntu:12.04 /bin/bash
```

## 拉取镜像

### docker run和默认的latest标签

```bash
sudo docker run -it --name next_container ubuntu /bin/bash
```

### 拉取fedora镜像

```bash
sudo docker pull fedora:20
```

### 查看fedora镜像

```bash
sudo docker images fedora
```

### 拉取带标签的fedora镜像

```bash
sudo docker pull fedora:21
```

## 查找镜像

### 查找镜像

```bash
sudo docker search puppet
```

### 拉取 macadmins/puppetmaster镜像

```bash
sudo docker pull macadmins/puppetmaster
```

### 从macadmins/puppetmaster镜像创建容器

```bash
sudo docker run -it macadmins/puppetmaster /bin/bash
facter
puppet --version
```

## 构建镜像

### 登陆到docker hub 

```bash
sudo docker login
```

### 创建一个要修改的定制容器

```bash
sudo docker run -it ubuntu /bin/bash
apt-get -yqq update
apt-get -y install apache2
```

### 提交定制容器

```bash
sudo docker commit 4aab3ce3cb76 user/pwd
```

### 提交一个新的定制容器

```bash
sudo docker commit -m 'a new custom image' -a 'dalong zheng' 4aab3ce3cb76 user/pwd:webserver
```

## 用docker file构建镜像

### 创建一个示例仓库

```bash
mkdir static_web
cd static_web
touch Dockerfile
```

### 第一个docker file

```bash
# Version: 0.0.1
FROM ubuntu:14.04
MAINTAINER zhengdalong 'dalong@example.com'
RUN apt-get update && apt-get install -y nginx
RUN echo 'hi ,i am in your container' > /usr/share/nginx/html/index.html
EXPOSE 80
```

### exec 格式的RUN命令

```bash
RUN ["apt-get","install","-y","nginx"]
```

### 基于docker file构建新镜像

```bash
cd static_web
sudo docker build -t="dalong/static_web" .
```

### 在构建时为镜像设置标签

```bash
sudo docker build -t="dalong/static_web:v1" .
```

### 从git仓库构建docker镜像

```bash
sudo docker build -t="dalong/static_web:v1" git@github.com:zhengdalong/docker-static_web
```

### 忽略dockerfile的构建缓存

```bash
sudo docker build --no-cache -t="dalong/static_web:v1" .
```

### ubuntu系统的docker file模版

```bash
FROM ubuntu
MAINTAINER zhengdalong 'dalong@example.com'
ENV REFRESHED_AT 2014-07-01
RUN apt-get -qq update
```

### fedora系统的docker file模版

```bash
FROM fedora:20
MAINTAINER zhengdalong 'dalong@example.com'
ENV REFRESHED_AT 2014-07-01
RUN yum -q makecache
```

## 查看新镜像

### 列出新的docker镜像 

```bash
sudo docker images dalong/static_web
```

### 使用docker history命令

```bash
sudo docker history ca02fd38313f
```

### 从新镜像启动一个容器

```bash
sudo docker run -d -p 80 --name static_web dalong/static_web:v1 nginx -g "daemon off;"
```

### docker port 命令

```bash
sudo docker port 8f1023633570 80
```

### 通过-p选项映射到特定端口

```bash
sudo docker run -d -p 8080:80 --name static_web dalong/static_web:v1 nginx -g "daemon off;"
```

### 绑定到特定的网络接口

```bash
sudo docker run -d -p 127.0.0.1:80:80 --name static_web dalong/static_web:v1 nginx -g "daemon off;"
```

### 绑定到特定的网络接口的随机端口

```bash
sudo docker run -d -p 127.0.0.1::80 --name static_web dalong/static_web:v1 nginx -g "daemon off;"
```

### 使用docker run命令对外公开端口

```bash
sudo docker run -d -P --name static_web dalong/static_web:v1 nginx -g 'daemon off;'
```

### 使用curl连接到容器

```bash
curl localhost:49154
```

## docker file指令

### CMD

#### 指定要运行的特定命令

```bash
sudo docker run -it dalong/static_web /bin/true
```

#### 使用CMD指令

```bash
CMD ["/bin/true"]
```

#### 给cmd指令传递参数

```bash
CMD ["/bin/bash","-l"]
```
注意cmd命令在docker file中只能存在一条，可被运行时命令覆盖，需要多条命令可以使用supervisor服务管理工具

### ENTRYPOINT

ENTRYPOINT命令可以理解为不可覆盖的CMD命令

#### 为entrypoint指定参数
```bash
ENTRYPOINT ['/usr/sbin/nginx','-g','daemon off;']
```

### WORKDIR

#### 使用workdir指令

```bash
WORKDIR /opt/webapp/db
RUN bundle install
WORKDIR /opt/webapp
ENTRYPOINT ['rackup']
```

#### 覆盖工作目录

```bash
sudo docker run -it -w /var/log ubuntu pwd
```

### ENV

#### 使用ENV设置多个环境变量

```bash
ENV RVM_PATH=/home/rvm RVM_ARCHFLAGS='-arch i386'
```

#### 在其他docker file指令中使用环境变量

```bash
ENV TARGET_DIR /opt/app
WORKDIR $TARGET_DIR
```

#### 运行时环境变量

```bash
sudo docker run -it -e 'WEB_PORT=8080' ubuntu env
```

### 指定USER和GROUP的各种组合

```bash
USER user
USER user:group
USER uid
USER uid:gid
USER user:gid
USER uid:group
```

### 使用VOLUME指令指定多个卷 

挂载文件系统

```bash
VOLUME ['/opt/project','/data']
```

### ADD

将构建环境下的文件复制到镜像中
支持从链接中复制
支持自动解压文件
自动创建目标路径
根据目标路径结尾是否有/判断源位置是文件还是目录
会使得构建缓存失效

```bash
ADD software.lic /opt/application/software.lic
ADD http://wordpress.org/latest.zip /root/wordpress.zip
ADD latest.tar.gz /var/www/wordpress
```

### COPY 

该命令和ADD相似，但不做文件提取和解压工作，只在构建上下文中复制本地文件
```bash
COPY conf.d/ /etc/apache2
```

### LABEL 

用于添加镜像元数据
最好在一行中添加，减少镜像层数
```bash
LABEL version='1.0'
LABEL location='new york' type='data center' role='web server'
```

#### STOPSIGNAL

用来设置停止容器时发送什么系统调用信号给容器

### ARG

#### 添加ARG指令
```bash
ARG build
ARG webapp_user=user
```

#### 使用ARG指令
```bash
docker build --build-arg build=1234 -t dalong/static_web
```

#### 预定义ARG变量

```bash
HTTP_PROXY
http_proxy
HTTPS_PROXY
https_proxy
FTP_PROXY
ftp_proxy
NO_PROXY
no_proxy
```

### ONBUILD
为镜像添加触发器
被其他镜像作为基础镜像的时候被触发
只能被子镜像触发，只可以被继承一次

#### 添加ONBUILD指令

```bash
ONBUILD ADD . /app/src
ONBUILD RUN cd /app/src && make
```

#### 新的docker镜像

```bash
FROM ubuntu
MAINTAINER zhengdalong 'dalong@example.com'
RUN apt-get update && apt-get install -y apache2
ENV APACHE_RUN_USER www-data
ENV APACHE_RUN_GROUP www-data
ENV APACHE_LOG_DIR /var/log/apache2
ONBUILD ADD . /var/www/
EXPOSE 80
ENTRYPOINT ["/usr/sbin/apache2"]
CMD ["-D","FOREGROUND"]
```

#### webapp的docker file

```bash
FROM dalong/apache2
MAINTAINER zhengdalong "dalong@example.com"
ENV APPLICATION_NAME webapp
ENV ENVIRONMENT development
```

## 将镜像推送到docker hub 

### 推送docker镜像
```bash
sudo docker push dalong/static_web
```

### 删除镜像
```bash
sudo docker rmi dalong/static_web
```

### 删除多个镜像
```bash
sudo docker rmi dalong/static_web dalong/apache2
```

### 删除所有镜像
```bash
sudo docker rmi 'docker images -a -q'
```

### 运行基于容器的registry
```bash
sudo docker run -p -d 5000:5000 registry:2
```

### 使用新registry为镜像打tag
```bash
sudo docker tag 281cc7ebf558 docker.example.com:5000/dalong/apache2
```

### 将镜像推送到新registry
```bash
sudo docker push docker.example.com:5000/dalong/apache2
```

### 从本地registry构建新的容器
```bash
sudo docker run -it docker.example.com:5000/dalong/apache2 /bin/bash
```







