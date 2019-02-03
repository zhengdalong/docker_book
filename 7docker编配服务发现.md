## docker compose

### 创建composeapp文件夹

```bash
mkdir composeapp
cd composeapp
touch Dockerfile
```

### app.py文件

```bash
from flask import Flask
from redis import Redis
import os

app = Flask(__name__)
redis = Redis(host="redis", port=6379)

@app.route('/')
def hello():
    redis.incr('hits')
    return 'Hello Docker Book reader! I have been seen {0} times'.format(int(redis.get('hits')))

if __name__ == "__main__":
    app.run(host="0.0.0.0", debug=True)
```

### requirements.txt文件

```bash
flask
redis
```

### composeapp dockerfile

```bash
FROM python:2.7
LABEL maintainer="james@example.com"
ENV REFRESHED_AT 2016-08-01

ADD . /composeapp

WORKDIR /composeapp

RUN pip install -r requirements.txt
```

### 构建composeapp应用

```bash
sudo docker build -t dalong/composeapp
```

### docker-compose.yml文件

```bash
version: '3'
services:
  web:
    image: dalong/composeapp
    command: python app.py
    ports:
     - "5000:5000"
    volumes:
     - .:/composeapp
  redis:
    image: redis
```

### 运行compose

```bash
cd composeapp
sudo docker-compose up
sudo docker-compose up -d
sudo docker-compose ps
sudo docker-compose logs
sudo docker-compose start
sudo docker-compose stop
sudo docker-compose rm
```

## consul 服务发现

### 创建目录保存consul的dockerfile

```bash
mkdir consul
cd consul
touch Dockerfile
```

### consul的dockerfile

```bash
FROM ubuntu:18.04
LABEL maintainer="james@example.com"
ENV REFRESHED_AT 2014-08-01

RUN apt-get -qq update
RUN apt-get -qq install curl unzip

ADD https://releases.hashicorp.com/consul/0.3.1/consul_0.3.1_linux_amd64.zip /tmp/consul.zip
RUN cd /usr/sbin && unzip /tmp/consul.zip && chmod +x /usr/sbin/consul && rm /tmp/consul.zip
ADD consul.json /config/

EXPOSE 8300 8301 8301/udp 8302 8302/udp 8400 8500 53/udp

VOLUME ["/data"]

ENTRYPOINT [ "/usr/sbin/consul", "agent", "-config-dir=/config" ]
CMD []
```

### consul.json配置文件

```bash
{
  "data_dir": "/data",
  "ui_dir": "/webui",
  "client_addr": "0.0.0.0",
  "ports": {
    "dns": 53
  },
  "recursor": "8.8.8.8"
}
```

### 构建consul镜像

```bash
sudo docker build -t dalong/consul .
```

### 执行一个本地consul节点

```bash
docker run -p 8500:8500 -p 53:53/udp -h node1 dalong/consul -server -bootstrap
```

### 设置公共ip地址

```bash
PUBLIC_IP='$(ifconfig eth0|awk -F '*|:' '/inet addr/{print $4}')'
```

### 获取docker0的IP地址

```bash
ip addr show docker0
```

### docker默认值

```bash
DOCKER_OPTS='--dns 8.8.8.8 --dns 8.8.4.4'
```

### 自启动主机新的默认值

```bash
DOCKER_OPTS='--dns 172.17.42.1 --dns 8.8.8.8 --dns-search service.consul'
```
重启docker守护进程
sudo service docker restart

### 启动具有自启动功能的consul节点

```bash
sudo docker run -d -h $HOSTNAME \
 -p 8300:8300 -p 8301:8301 \
 -p 8301:8301/udp -p 8302:8302 \
 -p 8302:8302/udp -p 8400:8400 \
 -p 8500:8500 -p 53:53/udp \
 --name larry_agent daong/consul \
 -server -advertise $PUBLIC_IP -bootstrap-expect 3
```

### 启动其余节点

```bash
sudo docker run -d -h $HOSTNAME \
 -p 8300:8300 -p 8301:8301 \
 -p 8301:8301/udp -p 8302:8302 \
 -p 8302:8302/udp -p 8400:8400 \
 -p 8500:8500 -p 53:53/udp \
 --name larry_agent daong/consul \
 -server -advertise $PUBLIC_IP -join $JOIN_IP
```

### 测试consul的dns服务

```bash
dig @172.17.42.1 consul.service.consul
```

### 启动 distributed_app

```bash
sudo docker run -h $HOSTNAME -d --name larry_distributed dalong/distributed_app
```

## docker swarm

### 拉取swarm镜像

```bash
sudo docker pull swarm
```

### 创建docker swarm

```bash
sudo docker run --rm swarm create
```

### 运行 swarm代理

```bash
sudo docker run -d swarm join  --addr=10.0.0.125:2375 token://b811b0bc438cb9a06fb68a25f1c9d8ab
```

### 列出swarm节点

```bash
docker run --rm swarm list token://b811b0bc438cb9a06fb68a25f1c9d8ab
```

### 启动swarm集群管理者

```bash
docker run -d -p 2380:2375 swarm manage token://b811b0bc438cb9a06fb68a25f1c9d8ab
```

### 在swarm集群中运行docker info命令

```bash
sudo docker -H tcp://localhost:2380 info
```

### 通过循环创建6个nginx容器

```bash
for i in 'seq 1 6'; do sudo docker -H tcp://localhost:2380 run -d --name www-$i -p 80 nginx;done
```

### swarm在集群中执行docker ps输出

```bash
docker -H tcp://localhost:2380 ps
```

## 过滤器
约束过滤器，亲和过滤器，依赖过滤器，端口过滤器，健康过滤器

### 运行docker守护进程时设置约束标签

```bash
sudo docker daemon --label datacenter=us-east1
```

### 启动容器时指定约束过滤器

```bash
sudo docker -H tcp://localhost:2380 run -e constraint:datacenter==us-east1 -d --name www-use1 -p 80 nginx
```

### 启动容器时在约束过滤器中使用正则表达式 

```bash
sudo docker -H tcp://localhost:2380 run -e constraint:datacenter==us-east* -d --name www-use1 -p 80 nginx
```

### 启动容器时指定亲和过滤器

```bash
sudo docker run -d --name www-use2 -e affinity:container==www-use1 nginx
```

### 启动容器时在亲和过滤器中使用不等于条件

```bash
sudo docker run -d --name db1 -e affinity:container!=www-use1 mysql
```

### 启动容器时在亲和过滤器中使用正则表达式

```bash
sudo docker run -d --name db1 -e affinity:container!=www-use* mysql
```

### 使用端口过滤器

```bash
sudo docker -H tcp://localhost:2380 run -d --name haproxy -p 80:80 haproxy
```










