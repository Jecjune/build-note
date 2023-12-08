
# 安装
```php
sudo apt-get update
```

由于 apt 源使用 HTTPS 以确保软件下载过程中不被篡改。因此，我们首先需要添加使用 HTTPS 传输的软件包以及 CA 证书。
```php
sudo apt-get install \
     apt-transport-https \
     ca-certificates \
     curl \
     gnupg \
     lsb-release
```

鉴于国内网络问题，强烈建议使用国内源，官方源请在注释中查看。为了确认所下载软件包的合法性，需要添加软件源的 GPG 密钥。
```php
curl -fsSL https://mirrors.aliyun.com/docker-ce/linux/debian/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```

然后，我们需要向 `sources.list` 中添加 Docker 软件源：
```php
echo \
  "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://mirrors.aliyun.com/docker-ce/linux/debian \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

更新 apt 软件包缓存，并安装 `docker-ce`。
```
sudo apt-get update
```
```
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

使 Docker 服务在每次重启时自动启动：
```
sudo systemctl enable docker
```
启动docker:
```
sudo systemctl start docker
```
测试是否正常:
```
docker run --rm hello-world
```



# 用户管理
默认情况下，`docker` 命令会使用 [Unix socket](https://en.wikipedia.org/wiki/Unix_domain_socket) 与 Docker 引擎通讯。而只有 `root` 用户和 `docker` 组的用户才可以访问 Docker 引擎的 Unix socket。出于安全考虑，一般 Linux 系统上不会直接使用 `root` 用户。因此，更好地做法是将需要使用 `docker` 的用户加入 `docker` 用户组。

建立docker组
```
sudo groupadd docker
```
将当前用户加入 `docker` 组：
```
sudo usermod -aG docker $USER
```