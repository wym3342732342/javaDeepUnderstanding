# 端口转发

### 1. 创建dev.project.forwarding文件

```shell
sudo vim /etc/pf.anchors/dev.project.forwarding
```

&emsp;&emsp;在idea.tomcat.forwarding添加以下命令：

```shell
rdr pass on lo0 inet proto tcp from any to 127.0.0.1 port 80 -> 127.0.0.1 port 3000
rdr pass on lo0 inet proto tcp from any to 127.0.0.1 port 443 -> 127.0.0.1 port 8080
```

### 2.创建pf-dev.conf文件

```shell
sudo vim /etc/pf-dev.conf 
```

&emsp;&emsp;pf-dev.conf添加以下命令：

```shell
rdr-anchor "forwarding"
load anchor "forwarding" from "/etc/pf.anchors/dev.project.forwarding"
```

### 3.启动端口转发功能

```shell
sudo pfctl -ef /etc/pf-dev.conf
```

&emsp;&emsp;如果你在终端看到以下提示，恭喜你成功启动：

```shell
pfctl: Use of -f option, could result in flushing of rules
present in the main ruleset added by the system at startup.
See /etc/pf.conf for further details.

No ALTQ support in kernel
ALTQ related functions disabled
pfctl: pf already enabled
```



### 4.关闭端口转发功能

```shell
sudo pfctl -d 
```

或者关闭全部

```shell
pfctl -F all -f /etc/pf.conf 
```

**注意事项：** 
&emsp;&emsp;重启mac，需要手动重启端口转发命令

