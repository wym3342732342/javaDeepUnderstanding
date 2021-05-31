部署项目服务器时，为了应对停电等情况影响正常web项目的访问，会把Docker容器设置为开机自动启动。

 

​    在使用docker run启动容器时，使用--restart参数来设置：

 

```
# docker run -m 512m --memory-swap 1G -it -p 58080:8080 --restart=always 
--name bvrfis --volumes-from logdata mytomcat:4.0 /root/run.sh
```

​    --restart具体参数值详细信息：

 

​    no - 容器退出时，不重启容器；

​    on-failure - 只有在非0状态退出时才从新启动容器；

​    always - 无论退出状态是如何，都重启容器；

 

​    还可以在使用on - failure策略时，指定Docker将尝试重新启动容器的最大次数。默认情况下，Docker将尝试永远重新启动容器。

```
# sudo docker run --restart=on-failure:10 redis
```

如果创建时未指定 --restart=always ,可通过update 命令

```
docker update --restart=always xxx
```