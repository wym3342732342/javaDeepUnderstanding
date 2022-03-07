# harbor安装部署

## 一、harbor简介

&emsp;&emsp;`Harbor`是构建企业级私有`docker`镜像的仓库的开源解决方案，它是`Docker Registry`的更高级封装，它除了提供友好的`Web UI`界面，角色和用户权限管理，用户操作审计等功能外，它还整合了`K8s`的插件(Add-ons)仓库，即Helm通过chart方式下载，管理，安装K8s插件，而`chartmuseum`可以提供存储`chart`数据的仓库【注:helm就相当于k8s的yum】。



&emsp;&emsp;另外它还整合了两个开源的安全组件，一个是`Notary`，另一个是`Clair`，这两个安全功能对于企业级私有仓库来说是非常具有意义的。

- Notary类似于私有CA中心;
- Clair则是容器安全扫描工具，它通过各大厂商提供的CVE漏洞库来获取最新漏洞信息，并扫描用户上传的容器是否存在已知的漏洞信息;





&emsp;&emsp;简单来说`harbor`就是`VMWare`公司提供的一个`docker`私有仓库构建程序。

&emsp;&emsp;支持多租户签名和认证 支持安全扫描和风险分析 这次日志审计 基于角色的访问控制 支持可扩展的`API`和`GUI Image replication between instances` 国际化做的很好(目前仅支持英文和中文)





## 二、安装部署

&emsp;&emsp;[harbor的github地址]([goharbor/harbor: An open source trusted cloud native registry project that stores, signs, and scans content. (github.com)](https://github.com/goharbor/harbor))

### 2.1 安装前的准备

1. docker

2. docker-compose

3. https证书

   ```shell
   mkdir -p /data/cert && cd /data/cert/
   
   ### 生成证书：希望能成功的方式
   
   vim harbor.conf
   # 文件内容
   [ req ]
   default_bits       = 4096
   distinguished_name = req_distinguished_name
    
   [ req_distinguished_name ]
   countryName                 = CN
   countryName_default         = CN
   stateOrProvinceName         = Beijing
   stateOrProvinceName_default = Beijing
   localityName                = Beijing
   localityName_default        = Beijing
   organizationName            = WZ
   organizationName_default    = WZ
   commonName                  = 192.168.1.115
   commonName_max              = 64
   commonName_default          = 192.168.1.115
   
   openssl genrsa -out harbor.key 4096
   
   openssl req \
     -new \
     -sha256 \
     -out harbor.csr \
     -key harbor.key \
     -config harbor.conf
     
   openssl x509 \
       -req \
       -days 3650 \
       -in harbor.csr \
       -signkey harbor.key \
       -out harbor.crt
   
   vim server.conf
   
   [ req ]
   default_bits       = 2048
   distinguished_name = req_distinguished_name
   req_extensions     = req_ext
    
   [ req_distinguished_name ]
   countryName                 = CN
   countryName_default         = CN
   stateOrProvinceName         = Beijing
   stateOrProvinceName_default = Beijing
   localityName                = Beijing
   localityName_default        = Beijing
   organizationName            = WZ
   organizationName_default    = WZ
   commonName                  = 192.168.1.115
   commonName_max              = 64
   commonName_default          = 192.168.1.115
    
   [ req_ext ]
   subjectAltName = @alt_names
   
   [alt_names]
   DNS.1   = 192.168.1.115
   IP      = 192.168.1.115
   
   openssl genrsa -out server.key 2048
   
   openssl req \
     -new \
     -sha256 \
     -out server.csr \
     -key server.key \
     -config server.conf
   
   openssl x509 \
     -req \
     -days 3650 \
     -CA harbor.crt \
     -CAkey harbor.key \
     -CAcreateserial \
     -in server.csr \
     -out server.pem\
     -extensions req_ext \
     -extfile server.conf
   
   ls
   ```
   
   



### 2.2 开始安装

1. 下载安装包：[Releases · goharbor/harbor (github.com)](https://github.com/goharbor/harbor/releases)

   - `Harbor offline installer`：包含harbor需要使用的镜像文件
   - `Harbor online installer`：

2. 下载后解压缩

   ```shell
   wget https://github.com/goharbor/harbor/releases/download/v1.10.10/harbor-offline-installer-v1.10.10.tgz
   tar zxf harbor-offline-installer-v1.10.10.tgz -C /data/
   ```

3. 进入目录

   ```shell
   cd /data/harbor/
   ```

4. 编辑`harbor.yml`

   ```shell
   vim harbor.yml
   
   ### 此处我们需要修改的是：hostname、https证书、admin密码
   
   # Configuration file of Harbor
   
   # The IP address or hostname to access admin UI and registry service.
   # DO NOT use localhost or 127.0.0.1, because Harbor needs to be accessed by external clients.
   hostname: ### 更换成本地ip
   
   # http related config
   http:
     # port for http, default is 80. If https enabled, this port will redirect to https port
     port: 80
   
   # https related config
   https:
     # https port for harbor, default is 443
     port: 443
     # The path of cert and key files for nginx
     certificate: /data/cert/harbor.crt # 或者是pem
     private_key: /data/cert/harbor.key # 这块就是私钥就行了
   
   # Uncomment external_url if you want to enable external proxy
   # And when it enabled the hostname will no longer used
   # external_url: https://reg.mydomain.com:8433
   
   # The initial password of Harbor admin
   # It only works in first time to install harbor
   # Remember Change the admin password from UI after launching Harbor.
   harbor_admin_password: admin # 默认密码
   
   # Harbor DB configuration
   database:
     # The password for the root user of Harbor DB. Change this before any production use.
     password: root123
     # The maximum number of connections in the idle connection pool. If it <=0, no idle connections are retained.
     max_idle_conns: 50
     # The maximum number of open connections to the database. If it <= 0, then there is no limit on the number of open connections.
     # Note: the default number of connections is 100 for postgres.
     max_open_conns: 100
   
   # The default data volume
   data_volume: /data
   
   # Harbor Storage settings by default is using /data dir on local filesystem
   # Uncomment storage_service setting If you want to using external storage
   # storage_service:
   #   # ca_bundle is the path to the custom root ca certificate, which will be injected into the truststore
   #   # of registry's and chart repository's containers.  This is usually needed when the user hosts a internal storage with self signed certificate.
   #   ca_bundle:
   
   #   # storage backend, default is filesystem, options include filesystem, azure, gcs, s3, swift and oss
   #   # for more info about this configuration please refer https://docs.docker.com/registry/configuration/
   #   filesystem:
   #     maxthreads: 100
   #   # set disable to true when you want to disable registry redirect
   #   redirect:
   #     disabled: false
   
   # Clair configuration
   clair:
     # The interval of clair updaters, the unit is hour, set to 0 to disable the updaters.
     updaters_interval: 12
   
   jobservice:
     # Maximum number of job workers in job service
     max_job_workers: 10
   
   notification:
     # Maximum retry count for webhook job
     webhook_job_max_retry: 10
   
   chart:
     # Change the value of absolute_url to enabled can enable absolute url in chart
     absolute_url: disabled
   
   # Log configurations
   log:
     # options are debug, info, warning, error, fatal
     level: info
     # configs for logs in local storage
     local:
       # Log files are rotated log_rotate_count times before being removed. If count is 0, old versions are removed rather than rotated.
       rotate_count: 50
       # Log files are rotated only if they grow bigger than log_rotate_size bytes. If size is followed by k, the size is assumed to be in kilobytes.
       # If the M is used, the size is in megabytes, and if G is used, the size is in gigabytes. So size 100, size 100k, size 100M and size 100G
       # are all valid.
       rotate_size: 200M
       # The directory on your host that store log
       location: /var/log/harbor
   
     # Uncomment following lines to enable external syslog endpoint.
     # external_endpoint:
     #   # protocol used to transmit log to external endpoint, options is tcp or udp
     #   protocol: tcp
     #   # The host of external endpoint
     #   host: localhost
     #   # Port of external endpoint
     #   port: 5140
   
   #This attribute is for migrator to detect the version of the .cfg file, DO NOT MODIFY!
   _version: 1.10.0
   
   # Uncomment external_database if using external database.
   # external_database:
   #   harbor:
   #     host: harbor_db_host
   #     port: harbor_db_port
   #     db_name: harbor_db_name
   #     username: harbor_db_username
   #     password: harbor_db_password
   #     ssl_mode: disable
   #     max_idle_conns: 2
   #     max_open_conns: 0
   #   clair:
   #     host: clair_db_host
   #     port: clair_db_port
   #     db_name: clair_db_name
   #     username: clair_db_username
   #     password: clair_db_password
   #     ssl_mode: disable
   #   notary_signer:
   #     host: notary_signer_db_host
   #     port: notary_signer_db_port
   #     db_name: notary_signer_db_name
   #     username: notary_signer_db_username
   #     password: notary_signer_db_password
   #     ssl_mode: disable
   #   notary_server:
   #     host: notary_server_db_host
   #     port: notary_server_db_port
   #     db_name: notary_server_db_name
   #     username: notary_server_db_username
   #     password: notary_server_db_password
   #     ssl_mode: disable
   
   # Uncomment external_redis if using external Redis server
   # external_redis:
   #   host: redis
   #   port: 6379
   #   password:
   #   # db_index 0 is for core, it's unchangeable
   #   registry_db_index: 1
   #   jobservice_db_index: 2
   #   chartmuseum_db_index: 3
   #   clair_db_index: 4
   
   # Uncomment uaa for trusting the certificate of uaa instance that is hosted via self-signed cert.
   # uaa:
   #   ca_file: /path/to/ca
   
   # Global proxy
   # Config http proxy for components, e.g. http://my.proxy.com:3128
   # Components doesn't need to connect to each others via http proxy.
   # Remove component from `components` array if want disable proxy
   # for it. If you want use proxy for replication, MUST enable proxy
   # for core and jobservice, and set `http_proxy` and `https_proxy`.
   # Add domain to the `no_proxy` field, when you want disable proxy
   # for some special registry.
   proxy:
     http_proxy:
     https_proxy:
     # no_proxy endpoints will appended to 127.0.0.1,localhost,.local,.internal,log,db,redis,nginx,core,portal,postgresql,jobservice,registry,registryctl,clair,chartmuseum,notary-server
     no_proxy:
     components:
       - core
       - jobservice
       - clair
   ```

5. 配置完成后

   ```shell
   ./prepare
   ```

6. 执行放到docker容器当中

   ```shell
   ./install.sh
   ```

   

### 2.3 常用管理命令

```shell
停止服务： docker-compose stop
开始服务： docker-compose start
重启服务：docker-compose restart
停止服务并删除容器：docker-compose down
启动服务并运行容器：docker-compose up
```













## 附：

### 1、docker login

```shell
docker login [-e|-email=""] [-p|--password=""] [-u|--username=""] [SERVER]

### 最简单的应用
docker login 127.0.0.1:5000
```

### 2、关于docker login无法登录私服的问题

```shell
# certificate signed by unknown authority
# 从官方文档提示，客户端要使用tls与Harbor通信，使用的还是自签证书，那么必须建立一个目录：/etc/docker/certs.d

### 在certs.d创建域名文件夹，将证书放入

pwd
# /etc/docker/certs.d/harbor.dev
 
cp /tmp/harbor.crt .
 
docker login harbor.dev
 
# Authenticating with existing credentials...
# WARNING! Your password will be stored unencrypted in /root/.docker/config.json.
# Configure a credential helper to remove this warning. See
# https://docs.docker.com/engine/reference/commandline/login/#credentials-store
 
# Login Succeeded
```















