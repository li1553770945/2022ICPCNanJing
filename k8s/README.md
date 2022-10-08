# 使用k8s部署

## 使用说明

### 1. 安装k8s

略

### 2. 安装nfs并创建一个共享目录

由于队伍照片会被放在容器内部，如果使用了多个domserver，就会出现在一个地方上传的图片另一个地方打不开的情况，因此需要将图片统一映射到一个文件夹上，这里用了nfs，你也可以改用其他的方式。

### 3. 修改参数

请修改以下参数：

domserver.yaml中：
- MYSQL_HOST:MYSQL主机
- MYSQL_PORT:MYSQL端口
- MYSQL_USER:MYSQL用户名
- MYSQL_DATABASE:MYSQL数据库名
- MYSQL_PASSWORD:MYSQL用户密码，和上面的用户名对应
- MYSQL_ROOT_PASSWORD:MYSQL root用户的密码


judgehost.yaml中：
- JUDGEDAEMON_PASSWORD:domserver中judgehost用户的密码
- DOMSERVER_BASEURL:domserver的地址

nfs-deployment.yaml中：

环境变量：
- NFS_SERVER：nfs服务器IP
- NFS_PATH:nfs共享目录

另外还需要修改volumes中的nfs的server和path。

```yaml
volumes:
    - name: nfs-client-root
        nfs:
        server: 192.168.1.226 #nfs服务器IP
        path: /nfsdata #nfs共享目录

```

### 4. 启动

```bash
kubectl create -f rbac.yaml
kubectl create -f nfs-deployment.yaml
kubectl create -f nfs-sc.yaml
kubectl create -f domserver-pvc.yaml
kubectl create -f domserver.yaml
kubectl create -f domservice.yaml
kubectl create -f judgehost.yaml

```

## 注意事项

1. judgehost设置了强制反亲和性，每台物理机器只能部署一个judgehost。
2. domserver挂载了图片文件夹，但是没有挂载自定义css和js文件夹，因此自定义css和js可能不生效。

## 文件说明

### 1. domserver.yaml

运行domserver的deployment。

### 2. domservice.yaml

domserver的服务，负载均衡和端口映射。

### 3. judgehost.yaml

运行评测机的deployment。理论上只需要这三个就行运行，但是问题是需要共享图片文件夹（队伍、学校图片等），后面的4个文件都是nfs相关的。

### 4. rbac.yaml

nfs的角色绑定。

### 5. nfs-deployment

运行storageclass的deployment。

### 6. nfs-sc.yaml

storageclass。

### 7. domserver-pvc.yaml

pvc的yaml文件。



## 一些问题

### 1. ErrImagePull
有一定概率pull失败，多等一会重试几次一般就OK了。