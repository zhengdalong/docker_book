## 确保docker已经就绪

### 查看docker程序是否正常工作

```bash
sudo docker info
```

## 运行我们的第一个容器

### docker run 命令

```bash
sudo docker run -it ubuntu /bin/bash
```

## 使用第一个容器

### 检查容器主机名

```bash
hostname
```

### 检查容器的/etc/hosts文件

```bash
cat /etc/hosts
```

### 检查容器接口

```bash
ip a
```

### 检查容器进程

```bash
ps -aux
```

### 在第一个容器中安装软件

```bash
apt-get update && apt-get install vim
```

### 列出docker容器

```bash
docker ps
```

## 容器命名

### 给容器命名 

```bash
 sudo docker run --name dalong_container -it ubuntu /bin/bash
```

## 重新启动已经停止的容器

### 启动已经停止运行的容器

```bash
 sudo docker start dalong_container
```

### 通过ID启动已经停止运行的容器

```bash
sudo docker start aa3f365f0f4e
sudo docker restart aa3f365f0f4e
```
docker create命令可以创建容器，但不运行

## 附着在容器上

### 附着到正在运行的容器 

```bash
sudo docker attach dalong_container
```

### 通过ID附着到正在运行的容器

```bash
sudo docker attach aa3f365f0f4e
```

## 创建守护式容器

### 创建长期运行的容器

```bash
sudo docker run --name daemon_dave -d ubuntu /bin/bash -c "while true; do echo hello world; sleep 1; done"
```

## 容器内部在干什么

### 获取守护容器的日志

```bash
sudo docker logs daemon_dave
```

### 跟踪守护容器的日志

```bash
sudo docker logs -f daemon_dave
```

### 跟踪守护容器最新日志

```bash
sudo docker logs -ft daemon_dave
```

## docker 日志驱动

### 在容器级别启动syslog

```bash
sudo docker run --log-driver="syslog" --name daemon_dave -d ubuntu /bin/bash -c "while true; do echo hello world; sleep 1; done"
```

## 查看容器内的进程

### 查看守护式容器的进程

```bash
sudo docker top daemon_dave
```

## docker统计信息

### docker stats 命令

```bash
sudo docker stats daemon_dave dalong_container
```

## 在容器内部运行进程

### 在容器中运行后台任务

```bash
sudo docker exec -d daemon_dave touch /etc/new_file
```

### 在容器内运行交互命令

```bash
sudo docker exec -it daemon_dave /bin/bash
```

## 停止守护式容器

### 停止正在运行的容器

```bash
sudo docker stop daemon_dave
sudo docker stop aa3f365f0f4e
```

### 显示最后x个容器

```bash
sudo docker ps -n x
```

## 自动重启容器

### 自动重启容器

```bash
sudo docker run --restart=always --name daemon_dave -d ubuntu /bin/sh -c "while true; do echo hello world; sleep 1; done"
```

### 为on-failure 指定count参数

```bash
sudo docker run --restart=on-failure:5 --name daemon_dave -d ubuntu /bin/sh -c "while true; do echo hello world; sleep 1; done"
```

## 深入容器

### 查看容器

```bash
sudo docker inspect daemon_dave
```

### 有选择的获取容器信息

```bash
sudo docker inspect --format='{{.state.running}}' daemon_dave
```

## 删除容器

### 删除容器

```bash
sudo docker rm aa3f365f0f4e
```

### 删除所有容器

```bash
sudo docker rm 'sudo docker ps -a -q'
```


