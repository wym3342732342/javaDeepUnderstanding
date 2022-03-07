# k8s安装ingress-nginx

一、基本信息

| 组件          | version              |      |
| ------------- | -------------------- | ---- |
| kubectl       | GitVersion:"v1.17.3" |      |
| docker        | 20.10.7              |      |
| ingress-nginx | 0.30.0               |      |

ingress-nginx  https://github.com/kubernetes/ingress-nginx/tree/nginx-0.30.0

参考视频 https://www.bilibili.com/video/BV1cK411p7MY?p=9

参考文档https://blog.csdn.net/YiSean96/article/details/105847045?utm_medium=distribute.pc_relevant.none-task-blog-2~default~baidujs_title~default-0.control&spm=1001.2101.3001.4242



二、安装

1.下载mandatory.yaml

https://github.com/kubernetes/ingress-nginx/blob/nginx-0.30.0/deploy/static/mandatory.yaml

执行部署

```sh
[root@master-dev ingress]# kubectl apply -f mandatory.yaml 
namespace/ingress-nginx created
configmap/nginx-configuration created
configmap/tcp-services created
configmap/udp-services created
serviceaccount/nginx-ingress-serviceaccount created
clusterrole.rbac.authorization.k8s.io/nginx-ingress-clusterrole created
role.rbac.authorization.k8s.io/nginx-ingress-role created
rolebinding.rbac.authorization.k8s.io/nginx-ingress-role-nisa-binding created
clusterrolebinding.rbac.authorization.k8s.io/nginx-ingress-clusterrole-nisa-binding created
deployment.apps/nginx-ingress-controller created
limitrange/ingress-nginx created

```

2.下载cloud-generic.yaml

https://github.com/kubernetes/ingress-nginx/blob/nginx-0.30.0/deploy/static/provider/cloud-generic.yaml

执行部署

````sh
[root@master-dev ingress]# kubectl apply -f cloud-generic.yaml 
service/ingress-nginx created
[root@master-dev ingress]# 

````

三、校验状态

多了ingress-nginx的命名空间

```sh
[root@master-dev ingress]# kubectl get ns
NAME                           STATUS   AGE
default                        Active   7d15h
ingress-nginx                  Active   14m
kube-node-lease                Active   7d15h
kube-public                    Active   7d15h
kube-system                    Active   7d15h
kubesphere-controls-system     Active   7d13h
kubesphere-monitoring-system   Active   7d13h
kubesphere-system              Active   7d14h
openebs                        Active   7d14h
project-petclinic              Active   3d18h
[root@master-dev ingress]# 

```

查看命名空间nginx下面的各项模块

```sh
[root@master-dev ingress]# kubectl get all -n ingress-nginx
NAME                                            READY   STATUS    RESTARTS   AGE
pod/nginx-ingress-controller-7f74f657bd-gk426   1/1     Running   0          16m

NAME                    TYPE           CLUSTER-IP     EXTERNAL-IP   PORT(S)                      AGE
service/ingress-nginx   LoadBalancer   10.96.167.88   <pending>     80:31668/TCP,443:32673/TCP   4m2s

NAME                                       READY   UP-TO-DATE   AVAILABLE   AGE
deployment.apps/nginx-ingress-controller   1/1     1            1           16m

NAME                                                  DESIRED   CURRENT   READY   AGE
replicaset.apps/nginx-ingress-controller-7f74f657bd   1         1         1       16m
[root@master-dev ingress]# 

```

EXTERNAL-IP状态为pengding，为异常。

解决方案：

删掉cloud-ger





















