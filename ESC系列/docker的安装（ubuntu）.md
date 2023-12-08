# 安装

本安装文档转载自：[容器与云|如何在 Ubuntu 22.04 LTS 中安装 Docker 和 Docker Compose (linux.cn)](https://linux.cn/article-14871-1.html#:~:text=%E5%9C%A8%20Ubuntu%2022.04%20LTS%20%E4%B8%AD%E5%AE%89%E8%A3%85%20Docker%201%201%E3%80%81%E6%9B%B4%E6%96%B0,%EF%BC%88%E9%80%89%E5%81%9A%EF%BC%89%20%E9%BB%98%E8%AE%A4%E6%83%85%E5%86%B5%E4%B8%8B%EF%BC%8CDocker%20%E5%AE%88%E6%8A%A4%E8%BF%9B%E7%A8%8B%E7%BB%91%E5%AE%9A%E5%88%B0%20Unix%20%E5%A5%97%E6%8E%A5%E5%AD%97%E8%80%8C%E4%B8%8D%E6%98%AF%20TCP%20%E7%AB%AF%E5%8F%A3%E3%80%82%20)

## Docker 依赖项
为了安装并配置 Docker ，你的系统必须满足下列最低要求：

1. 64 位 Linux 或 Windows 系统
2. 如果使用 Linux ，内核版本必须不低于 3.10
3. 能够使用 `sudo` 权限的用户
4. 在你系统 BIOS 上启用了 VT（虚拟化技术）支持 on your system BIOS（参考: [如何查看 CPU 支持 虚拟化技术（VT）](https://ostechnix.com/how-to-find-if-a-cpu-supports-virtualization-technology-vt/)）
5. 你的系统应该联网

在 Linux ，在终端上运行以下命令验证内核以及架构详细信息：
```
uname -a
```

## 在 Ubuntu 22.04 LTS 中安装 Docker
#### 1、更新 Ubuntu
```php
sudo apt update && sudo apt upgrade && sudo apt full-upgrade
```
#### 2、添加 Docker 库
安装必要的证书并允许 apt 包管理器使用以下命令通过 HTTPS 使用存储库：
```php
sudo apt install apt-transport-https ca-certificates curl software-properties-common gnupg lsb-release
```
运行下列命令添加 Docker 的官方 GPG 密钥：
```php
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```
添加 Docker 官方库：
```php
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```
使用命令更新 Ubuntu 源列表：
```php
sudo apt update
```
#### 3、安装 Docker
运行下列命令在 Ubuntu 22.04 LTS 服务器中安装最新 Docker CE：
```php
sudo apt install docker-ce docker-ce-cli containerd.io docker-compose-plugin
```

当然你也可以安装其他版本 Docker 。运行下列命令检查可以安装的 Docker 版本：
```php
apt-cache madison docker-ce
```
例如，安装 **5:20.10.16~ 3-0 ~ubuntu-jammy** 这个版本，运行：
```php
sudo apt install docker-ce=5:20.10.16~3-0~ubuntu-jammy docker-ce-cli=5:20.10.16~3-0~ubuntu-jammy containerd.io
```


安装完成后，运行如下命令验证 Docker 服务是否在运行：
```php
systemctl status docker
```
如果没有运行，运行以下命令运行 Docker 服务：
```php
sudo systemctl start docker
```
可以使用以下命令查看已安装的 Docker 版本：
```php
sudo docker version
```

使 Docker 服务在每次重启时自动启动：
```php
sudo systemctl enable docker
```

#### 4、测试 Docker
测试 Docker 是否运行正常：
```php
sudo docker run hello-world
```
上述命令会下载一个 Docker 测试镜像，并在容器内执行一个 “hello_world” 样例程序。如果你能看到***Hello from Docker!*** 的输出样例就表示docker运行成功了.

#### 5、作为非 root 用户运行 Docker （选做）
默认情况下，Docker 守护进程绑定到 Unix 套接字而不是 TCP 端口。由于 **Unix 套接字由 root 用户拥有**，Docker 守护程序将仅以 root 用户身份运行。因此，普通用户无法执行大多数 Docker 命令。

如果你想要在 Linux 中作为非 root 用户运行 Docker ，参考下方链接：

- [如何在 Linux 中作为非 root 用户运行 Docker](https://ostechnix.com/how-to-run-docker-as-non-root-user-in-linux/)

我个人不这样做也**不推荐**你这么做。如果你不会在互联网上暴露你的系统，那没问题。然而，不要在生产系统中以非 root 用户身份运行 Docker 。

## 在 Ubuntu 中安装 Docker Compose(选做)
**Docker Compose** 是一个可用于定义和运行多容器 Docker 应用程序的工具。使用 Compose，你可以使用 Compose 文件来配置应用程序的服务。然后，使用单个命令，你可以[[docker的基本指令#^c97484|从配置中创建和启动所有服务]]。

下列任何方式都可以安装 Docker Compose 。

#### 方式 1、使用二进制文件安装 Docker Compose
从 [这里](https://github.com/docker/compose/releases) 下载最新 Docker Compose 。
运行下列命令安装最新稳定的 Docker Compose 文件：
```php
sudo curl -L "https://github.com/docker/compose/releases/download/v2.6.1/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
```
如果有更新版本，只需要将上述命令中的 **v2.6.1** 替换为最新的版本号即可。请不要忘记数字前的 **"v"** 。最后，使用下列命令赋予二进制文件可执行权限：
```php
sudo chmod +x /usr/local/bin/docker-compose
```
运行下列命令检查安装的 Docker Compose 版本：
```php
docker-compose version
```

#### 方式 2、使用 Pip 安装 Docker Compose
或许，我们可以使用 **Pip** 安装 Docker Compose 。Pip 是 Python 包管理器，用来安装使用 Python 编写的应用程序。

参考下列链接安装 Pip 。

- [如何使用 Pip 管理 Python 包](https://ostechnix.com/manage-python-packages-using-pip/)

安装 Pip 后，运行以下命令安装 Docker Compose。下列命令对于所有 Linux 发行版都是相同的！
```php
pip install docker-compose
```
安装 Docker Compose 后，使用下列命令检查版本：
```php
docker-compose --version
```

