# 使用Docker或k8s部署domjudge


如果有条件更建议使用[k8s部署](./k8s/)，以下内容为使用docker部署。
## step1 启动server

首先修改`DOMServer/Dockerfile`数据库地址、端口、数据库名、root密码、常规账户密码，然后

```
cd DOMServer
docker build -t domserver .
docker run -p 80:80 -d {build出来的镜像id}
```

## step2 查看相关密码

使用
```
docker exec -it domserver cat /opt/domjudge/domserver/etc initial_admin_password.secret
```
可以看到初始的admin密码，需要记下。

使用
```
docker exec -it domserver cat /opt/domjudge/domserver/etc/restapi.secret
```

可以看到judgehost初始密码，在网页里面可以修改，即judgehost用户的密码，因此可以不记。

## step3 启动judgehost

修改`JudgeHost/Dockerfile`中domserver的地址和judgehost用户的密码。（注：该密码可以在admin里面修改，修改judgehost用户的密码就行）

```
cd JudgeHost
docker build -t judgehost .
docker run --privileged -v /sys/fs/cgroup:/sys/fs/cgroup:ro  --hostname judgedaemon-0  {build出来的镜像id}
```

如果是本机记得加上`--network host`

## 遇到的问题和解决办法

1. 编译日志输出`unable to resolve host judgedaemon-0: Name or service not known`。
   

无关紧要，在judgehost的host文件里配置一下应该就行。


   