# K8s使用harbor

## 一、创建secret

```shell
 kubectl create secret docker-registry regcred --docker-server=10.11.67.119 --docker-username=victory --docker-password=QianFeng@123
 
 
 kubectl create secret docker-registry docker-harbor --docker-server=192.168.1.115 --docker-username=admin --docker-password=02269253256Ww* --docker-email=kingbb0630@icloud.com –-namespace=cnooc

注：docker-harbor：secret的名称
–docker-server：Harbor仓库地址
–docker-username：Harbor仓库登录用户
–docker-password：Harbor仓库登录密码
–docker-email：接收邮件
–namespace：也可以指定命名空间如果不指定的话默认是default


 kubectl create secret docker-registry docker-harbor --docker-server=192.168.1.115 --docker-username=admin --docker-password=02269253256Ww* --docker-email=kingbb0630@icloud.com
```

