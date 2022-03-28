centos7 部署harbor

```sh
# 离线形式安装harbor私有镜像仓库

## 创建目录及下载harbor离线包
cd /opt/
wget https://github.com/goharbor/harbor/releases/download/v2.2.0/harbor-offline-installer-v2.2.0.tgz
#下载较慢，可以手动下载后上传

tar xf harbor-offline-installer-v2.2.0.tgz

## 创建harbor访问域名证书
mkdir /opt/ssl
cd /opt/ssl
openssl genrsa -out tls.key 2048
openssl req -new -x509 -key tls.key -out tls.cert -days 3650 -subj /CN=*.wz-harbor.com
```



修改配置文件，最后的配置文件如下

```sh
[root@harbor harbor]# cat harbor.yml |grep -v '#'

hostname: harbor.yb.com

http:
  port: 80

https:
  port: 443
  certificate: /opt/ssl/tls.cert
  private_key: /opt/ssl/tls.key

harbor_admin_password: Yb123@ops

database:
  password: root123
  max_idle_conns: 50
  max_open_conns: 1000

data_volume: /home/data

trivy:
  ignore_unfixed: false
  skip_update: false
  insecure: false

jobservice:
  max_job_workers: 10

notification:
  webhook_job_max_retry: 10

chart:
  absolute_url: disabled

log:
  level: info
  local:
    rotate_count: 50
    rotate_size: 200M
    location: /var/log/harbor

_version: 2.2.0


proxy:
  http_proxy:
  https_proxy:
  no_proxy:
  components:
    - core
    - jobservice
    - trivy

```



安装docker-compose

> 从二进制安装k8s项目的bin目录拷贝过来
> scp /etc/kubeasz/bin/docker-compose 10.0.1.204:/usr/bin/
> 也可以在docker官方进行下载
> https://docs.docker.com/compose/install/
> ##################################################

```sh
sudo curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

sudo chmod +x /usr/local/bin/docker-compose

sudo ln -s /usr/local/bin/docker-compose /usr/bin/docker-compose

docker-compose --version
docker-compose version 1.29.2, build 5becea4c
```

安装docker

```
1. 安装依赖包
#yum install -y yum-utils device-mapper-persistent-data lvm2 wget
#yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo

2. 安装docker
#yum install -y docker-ce docker-ce-cli containerd.io
3.配置加速器
#mkdir -p /etc/docker

#sudo tee /etc/docker/daemon.json <<-'EOF'
{
  "registry-mirrors": ["https://8vtfw82h.mirror.aliyuncs.com"]
}
EOF
3.启动docker
#systemctl daemon-reload && systemctl start docker && systemctl enable docker
```



开始安装harbor

```sh
cd /opt/harbor
./install.sh

```

查看状态

```sh
[root@harbor harbor]# docker-compose ps
      Name                     Command                  State                              Ports                        
------------------------------------------------------------------------------------------------------------------------
harbor-core         /harbor/entrypoint.sh            Up (healthy)                                                       
harbor-db           /docker-entrypoint.sh            Up (healthy)                                                       
harbor-jobservice   /harbor/entrypoint.sh            Up (healthy)                                                       
harbor-log          /bin/sh -c /usr/local/bin/ ...   Up (healthy)   127.0.0.1:1514->10514/tcp                           
harbor-portal       nginx -g daemon off;             Up (healthy)                                                       
nginx               nginx -g daemon off;             Up (healthy)   0.0.0.0:80->8080/tcp,:::80->8080/tcp,               
                                                                    0.0.0.0:443->8443/tcp,:::443->8443/tcp              
redis               redis-server /etc/redis.conf     Up (healthy)                                                       
registry            /home/harbor/entrypoint.sh       Up (healthy)                                                       
registryctl         /home/harbor/start.sh            Up (healthy)                                                       
[root@harbor harbor]# 

```



遇到关于证书的报错

```sh
[root@harbor harbor]# docker push harbor.yb.com/library/nginx:1.18.0-alpine
The push refers to repository [harbor.yb.com/library/nginx]
Get https://harbor.yb.com/v2/: x509: certificate signed by unknown authority

```

解决方案

```
sudo vi /etc/docker/daemon.json
加入insecure-registries
{  
   "insecure-registries":["私库地址"]
}

重启docker

sudo systemctl restart docker

## 重启harbor
docker-compose down -v
docker-compose up -d
docker ps|grep harbor

```



用浏览器登录harbor

https://harbor.yb.com/
admin
Yb123@ops

![image-20210727143048189](C:\Users\sxxzq\AppData\Roaming\Typora\typora-user-images\image-20210727143048189.png)





docker登录harbor

```sh
1.把证书从harbor拷贝到docker客户端
#mkdir -p /etc/docker/certs.d/harbor.yb.com

# scp harbor.yb.com:/opt/ssl/tls.cert /etc/docker/certs.d/harbor.yb.com/ca.crt
The authenticity of host 'harbor.yb.com (192.168.2.21)' can't be established.
ECDSA key fingerprint is SHA256:FMsa4dKCyxfq7MnhTACh2UPpuFMf504XvALNDKR0j/s.
ECDSA key fingerprint is MD5:16:35:54:8c:f7:b3:ca:95:6d:af:94:fd:6d:63:ac:f1.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'harbor.yb.com,192.168.2.21' (ECDSA) to the list of known hosts.
root@harbor.yb.com's password: 
tls.cert                                                                                           100% 1090    64.4KB/s   00:00    
[root@node02-dev docker]# ls
certs.d  daemon.json  key.json

2.登录
[root@node02-dev docker]# docker login harbor.yb.com
Username: dev
Password: Yb123@ops
WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
Configure a credential helper to remove this warning. See
https://docs.docker.com/engine/reference/commandline/login/#credentials-store

Login Succeeded
[root@node02-dev docker]# 

```



从docker上传镜像（先登录）

```sh
1.打标签
#docker tag yb-user harbor.yb.com/dev-v3/yb-user

2.登录
# docker push harbor.yb.com/dev-v3/yb-user
Using default tag: latest
The push refers to repository [harbor.yb.com/dev-v3/yb-user]
c77788fca844: Pushed 
e8144cfbd890: Pushed 
022e61f1a07d: Pushed 
91fef57e43c2: Pushed 
86c7f9599ba1: Pushed 
d148a6d68ba3: Pushed 
50644c29ef5a: Pushed 
latest: digest: sha256:134b7d39059c9892fadefed8fe202280930365e8c8a4a1a0ced6abbdd5ed029b size: 1793

```





报错

```
[root@harbor system]# docker-compose down -v
ERROR: 
        Can't find a suitable configuration file in this directory or any
        parent. Are you in the right directory?

        Supported filenames: docker-compose.yml, docker-compose.yaml, compose.yml, compose.yaml

#必须在/opt/harbor目录下执行

[root@harbor system]# cd /opt/harbor
[root@harbor harbor]# ls
common  common.sh  docker-compose.yml  harbor.v2.2.0.tar.gz  harbor.yml  harbor.yml.tmpl  install.sh  LICENSE  prepare
[root@harbor harbor]# docker-compose down -v
Stopping harbor-portal ... done
Stopping harbor-log    ... done
Removing nginx             ... done
Removing harbor-jobservice ... done
Removing harbor-core       ... done
Removing registryctl       ... done
Removing harbor-db         ... done
Removing harbor-portal     ... done
Removing redis             ... done
Removing registry          ... done
Removing harbor-log        ... done
Removing network harbor_harbor
[root@harbor harbor]# 

```

