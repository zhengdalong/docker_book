## 配置docker守护进程

### 修改docker守护进程的网络

```bash
sudo docker daemon -H tcp://0.0.0.0:2375
```

### 使用DOCKER_HOST 环境变量

```bash
export DOCKER_HOST="tcp://0.0.0.0:2375"
```

### 将docker守护进程绑定到非默认套接字

```bash
sudo docker daemon -H unix://home/docker/docker.sock
```

### 将docker进程绑定到多个地址

```bash
sudo docker daemon -H tcp://0.0.0.0:2375 -H unix://home/docker/docker.sock
```

### 开启docker守护进程的调试模式

```bash
sudo docker daemon -D
```
## 检查docker守护进程是否正在运行

### 检查docker守护进程的状态

```bash
sudo status docker
```

### 用upstart启动和停止docker守护进程

```bash
 sudo stop docker
 sudo start docker
```

### 在redhat和fedora中启动和停止docker

```bash
sudo service docker stop
sudo service docker statrt
```

## 升级docker

```bash
sudo apt-get update
sudo apt-get install docker-engine
```