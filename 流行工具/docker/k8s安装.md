# Minikube安装K8S

### 1、启动前的准备

```shell
#0、安装docker-ce
#1、下载安装包
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
#2、安装
sudo install minikube-linux-amd64 /usr/local/bin/minikube
```



### 2、启动

> 前置：
>
> ```shell
> cd /etc
> 
> chmod 777 sudoers
> 
> vi sudoers
> 
> # 添加制定用户权限
> 
> chmod 440 sudoers
> ```

**输入命令并启动：**

```shell
minikube start --driver=docker
minikube config set driver docker
```



### 3、安装kubectl

1. 下载最新发行版

   ```shell
   curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
   ```

2. 下载校验文件

   ```shell
   curl -LO "https://dl.k8s.io/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl.sha256"
   ```

3. 校验

   ```shell
   echo "$(<kubectl.sha256) kubectl" | sha256sum --check
   ```

4. 安装

   ```shell
   sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
   ```



### 4、配置kubectl与集群交互

```shell
kubectl get po -A
minikube kubectl -- get po -A
minikube dashboard
```



