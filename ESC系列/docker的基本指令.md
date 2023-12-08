
***docker的PGID=999***

**查看当前正在运行的容器**：
```php
 docker ps 
```
加上-a后才能看到停止的容器.

**创建一个容器**:
```php
docker run -d --name <待运行容器的名称> 
```
可选参数:
-p <宿主机端口:映射到容器中的端口> 
-v <宿主机的待挂载目录:容器中的目录> <系统镜像>:<版本，默认用latest>
-e <参数>=<值>   (一般用来设置PUID和PGID,UMASK; UMASK为创建的文件权限,具体参数:022对应755,000对应777)

**删除一个容器：**
```php
docker rm -f <容器名称或容器ID>
```

**运行一个容器：**
```php
docker start <容器名称>
```

**停止一个容器：**
```php
docker stop <容器名称>
```

**重启一个容器：** 
```php
docker restart <容器名称>
```
**进入一个容器：** 
```php
docker exec -it <容器名称或容器ID> <shell>
```
**查看容器日志:**
```php
docker logs frps
```

**查看当前容器镜像：**  
```php
docker images
```
**删除一个容器镜像：**
```php
docker rm <容器镜像ID 或者 镜像名称:版本>
```

**搜索镜像并拉取：**
```php
docker search <镜像名>
```
```php
docker pull <镜像名>:<版本>
```

**自建镜像：**
使用标准镜像构建配置文件 dockerfile 进行编辑，然后使用 
```
docker build -t <新建的镜像名> <dockerfile的位置>
```  
即可将自建的镜像加入镜像库中

## docker compose：

需要通过配置文件去调用,下面是一个配置文件例程:
docker-compose.yml ^c97484
```
version: '2.1'
services:
    <服务名称>：
        image:<镜像：版本>
        container_name: <容器名称>
        ports:
            - <宿主机端口:容器端口>
        volumes:
            - <宿主机目录:容器的目录>
        networks:
            - host1-network     (这里和下面network的设置对应)
    <服务名称>：
            ...
networks:
    host1-network:
        driver:bridge
        ipam:
            driver: default
            config:
                - subnet:<网段/24   如：192.168.11.0/24，24表示是24位的网段,容器只会在设定的网段中生成ip地址>
                  gateway:<网关 如：192.168.11.11.254>
```

如图所示，docker-compose可以指定网段和ip，如果容器创建在同一个networks中，那么容器间是可以直接通信的。

在同等目录下，执行
```php
docker-compose up -d 
```
即可通过yaml文件同时启动容器。

其他指令类似与docker,例如：docker-compose stop 等

