## 使用docker测试静态网站

### sample网站的初始dockerfile

#### 为nginx dockerfile创建一个目录

```bash
mkdir sample
cd sample
touch Dockerfile
```

#### 获取nginx配置文件

```bash
mkdir nginx && cd nginx
wget https://raw.githubusercontent.com/jamtur01/dockerbook-code/master/code/5/sample/nginx/global.conf
wget https://raw.githubusercontent.com/jamtur01/dockerbook-code/master/code/5/sample/nginx/nginx.conf
```

#### 网站测试的基本dockerfile

```bash
FROM ubuntu:18.04
LABEL maintainer="dalong@example.com"
ENV REFRESHED_AT 2014-06-01

RUN apt-get -yqq update && apt-get -yqq install nginx

RUN mkdir -p /var/www/html/website
ADD nginx/global.conf /etc/nginx/conf.d/
ADD nginx/nginx.conf /etc/nginx/

EXPOSE 80
```

### 构建sample网站和nginx镜像

#### 构建新的nginx镜像

```bash
sudo docker build -t='dalong/nginx' .
```

#### 展示nginx镜像的构建历史

```bash
sudo docker history dalong/nginx
```

#### 构建第一个nginx容器

```bash
sudo docker run -d -p 80 --name website -v $PWD/website:/var/www/html/website dalong/nginx nginx
```

#### 控制卷的写状态

```bash
sudo docker run -d -p 80 --name website -v $PWD/website:/var/www/html/website:ro dalong/nginx nginx
```

## 使用docker构建并测试web应用程序

### 构建sinatra应用程序

#### 为测试web应用程序创建目录

```bash
mkdir -p sinatra
cd sinatra
```

#### 测试用web应用程序的dockerfile

```bash
FROM ubuntu:18.04
LABEL maintainer="james@example.com"
ENV REFRESHED_AT 2014-06-01

RUN apt-get -qq update && apt-get -qq install ruby ruby-dev build-essential redis-tools
RUN gem install --no-rdoc --no-ri sinatra json redis

RUN mkdir -p /opt/webapp

EXPOSE 4567

CMD [ "/opt/webapp/bin/webapp" ]
```

#### 下载sinatra web应用程序

```bash
cd sinatra
ls -l webapp
```

#### sinatra app.rb源代码

```bash
require "rubygems"
require "sinatra"
require "json"

class App < Sinatra::Application

  set :bind, '0.0.0.0'

  get '/' do
    "<h1>DockerBook Test Sinatra app</h1>"
  end

  post '/json/?' do
    params.to_json
  end

end
```

#### 启动第一个sinatra容器

```bash
sudo docker run -d -p 4567 --name webapp -v $PWD/webapp:/opt/webapp dalong/sinatra
```

#### 查看容器

```bash
sudo docker logs -f webapp
sudo docker top webapp
sudo docker port webapp 4567
```

#### 测试sinatra应用程序

```bash
curl -i -H 'Accept: application/json' -d 'name=Foo&status=Bar' http://localhost:49160/json
```

### 扩展sinatra应用程序来使用redis

#### app.rb 代码

```bash
require "rubygems"
require "sinatra"
require "json"
require "redis"

class App < Sinatra::Application

      redis = Redis.new(:host => 'db', :port => '6379')

      set :bind, '0.0.0.0'

      get '/' do
        "<h1>DockerBook Test Redis-enabled Sinatra app</h1>"
      end

      get '/json' do
        params = redis.get "params"
        params.to_json
      end

      post '/json/?' do
        redis.set "params", [params].to_json
        params.to_json
      end
end
```

### 构建redis数据库镜像

#### 创建目录

```bash
mkdir -p sinatra/redis
cd sinatra/redis
```

#### redis镜像代码

```bash
FROM ubuntu:18.04
LABEL maintainer="james@example.com"
ENV REFRESHED_AT 2014-06-01

RUN apt-get -qq update && apt-get -qq install redis-server redis-tools

EXPOSE 6379

ENTRYPOINT ["/usr/bin/redis-server" ]
CMD []
```

#### 构建redis镜像,启动容器

```bash
sudo docker build -t dalong/redis .
sudo docker run -d -p 6379 --name redis dalong/redis
```

#### 测试redis链接

```bash
redis-cli -h 127.0.0.1 -p 49161
```

### docker内部联网

内部联网方案在重启容器时会改变IP地址，不可硬编码，不适用

#### 相关命令

```bash
ip a show docker0
sudo docker run -it ubuntu /bin/bash
ip a show eth0
apt-get install traceroute
traceroute google.com
sudo iptables -t nat -L -n
sudo docker inspect redis
redis-cli -h 172.17.0.18
sudo docker restart redis
```

### docker networking

#### 创建docker网络

```bash
sudo docker network create app
```

#### 查看新创建的app网络

```bash
sudo docker network inspect app
```

#### docker network ls 命令

#### 在docker网络中创建redis容器

```bash
sudo docker run -d --net=app --name db dalong/redis
```

#### 链接redis容器

```bash
cd sinatra
sudo docker run -p 4567 --net=app --name webapp_redis -it -v $PWD/webapp_redis:/opt/webapp dalong/sinatra /bin/bash
```

#### 检查webapp_redis容器的/etc/hosts文件

#### ping db.app

#### 启动启用了redis的sinatra应用程序

```bash
nohup /opt/webapp/bin/webapp &
```

### 将已有容器链接到docker网络

```bash
sudo docker network connect app db2
```

### 从网络中断开一个容器

```bash
sudo docker network disconnect app db2
```

### 通过docker链接链接容器

#### 链接redis容器

```bash
sudo docker run -p 4567 --name webapp --link redis:db -it -v $PWD/webapp_redis:/opt/webapp dalong/sinatra /bin/bash
```

## docker 持续集成

### jenkins dockerfile

```bash
FROM jenkins
MAINTAINER james@example.com
ENV REFRESHED_AT 2016-06-01

USER root
RUN apt-get -qq update && apt-get install -qq sudo
RUN echo "jenkins ALL=NOPASSWD: ALL" >> /etc/sudoers
RUN wget http://get.docker.com/builds/Linux/x86_64/docker-latest.tgz
RUN tar -xvzf docker-latest.tgz
RUN mv docker/* /usr/bin/

USER jenkins
RUN /usr/local/bin/install-plugins.sh junit git git-client ssh-slaves greenballs chucknorris ws-cleanup
```

### dockerjenkins.sh脚本

```bash
#!/bin/bash

# First, make sure that cgroups are mounted correctly.
CGROUP=/sys/fs/cgroup

[ -d $CGROUP ] ||
  mkdir $CGROUP

mountpoint -q $CGROUP ||
  mount -n -t tmpfs -o uid=0,gid=0,mode=0755 cgroup $CGROUP || {
    echo "Could not make a tmpfs mount. Did you use -privileged?"
    exit 1
  }

# Mount the cgroup hierarchies exactly as they are in the parent system.
for SUBSYS in $(cut -d: -f2 /proc/1/cgroup)
do
  [ -d $CGROUP/$SUBSYS ] || mkdir $CGROUP/$SUBSYS
  mountpoint -q $CGROUP/$SUBSYS ||
    mount -n -t cgroup -o $SUBSYS cgroup $CGROUP/$SUBSYS
done

# Now, close extraneous file descriptors.
pushd /proc/self/fd
for FD in *
do
  case "$FD" in
  # Keep stdin/stdout/stderr
  [012])
    ;;
  # Nuke everything else
  *)
    eval exec "$FD>&-"
    ;;
  esac
done
popd

docker daemon &
exec java -jar /opt/jenkins/jenkins.war
```

### 运行docker-jenkins镜像

```bash
sudo docker run -p 8080:8080 --name dalong_jenkins --privileged -d jenkins
```

### 用于jenkins作业的docker shell脚本

```bash
# Build the image to be used for this job.
IMAGE=$(sudo docker build . | tail -1 | awk '{ print $NF }')

# Build the directory to be mounted into Docker.
MNT="$WORKSPACE/.."

# Execute the build inside Docker.
CONTAINER=$(sudo docker run -d -v $MNT:/opt/project/ $IMAGE /bin/bash -c 'cd /opt/project/workspace && rake spec')

# Attach to the container so that we can see the output.
sudo docker attach $CONTAINER

# Get its exit code as soon as the container stops.
RC=$(sudo docker wait $CONTAINER)

# Delete the container we've just used.
sudo docker rm $CONTAINER

# Exit with the same value as that with which the process exited.
exit $RC
```

### 用于测试作业的docker file

```bash
FROM ubuntu:18.04
MAINTAINER dalong 'dalong@qq.com'
ENV REFRESHED_AT 2014-06-01
RUN apt-get update
RUN apt-get -y install ruby rake
RUN gem install --no-rdoc --no-ri rspec ci_reporter_rspec
```

### jenkins多配置脚本

```bash
# Build the image to be used for this run.
cd $OS && IMAGE=$(sudo docker build . | tail -1 | awk '{ print $NF }')

# Build the directory to be mounted into Docker.

MNT="$WORKSPACE/.."

# Execute the build inside Docker.
CONTAINER=$(sudo docker run -d -v "$MNT:/opt/project" $IMAGE /bin/bash -c "cd /opt/project/$OS && rake spec")

# Attach to the container's streams so that we can see the output.
sudo docker attach $CONTAINER

# As soon as the process exits, get its return value.
RC=$(sudo docker wait $CONTAINER)

# Delete the container we've just used to free disk space.
sudo docker rm $CONTAINER

# Exit with the same value that the process exited with.
exit $RC
```




















