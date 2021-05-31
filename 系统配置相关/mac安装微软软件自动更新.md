### 1、去除AutoUpdate权限

```shell
cd /Library/Application\ Support/Microsoft/MAU2.0
sudo chmod 000 Microsoft\ AutoUpdate.app/

## 恢复权限：默认权限为755 drwxr-xr-x
sudo chmod 755 Microsoft\ AutoUpdate.app/
```



```shell
444 r--r--r-- ： 所有用户都只有读权限
600 rw------- ： 文件所有者具有读、写权限，其他用户没有权限
644 rw-r--r-- ： 文件所有者具有读写权限，同组用户具有读权限，其他用户具有读权限
666 rw-rw-rw- ：文件所有者，同组用户，其他用户都具有读写权限，没有执行权限
700 rwx------ ： 文件所有者具有读写执行权限，同组用户其他用户均没有任何权限
744 rwxr--r-- ： 文件所有者具有读写执行权限，同组用户和其他用户只有读权限
755 rwxr-xr-x ： 文件所有者具有读、写、执行权限，同组用户和其他用户具有读、执行权限
777 rwxrwxrwx ： 全部用户都用全权限
```

