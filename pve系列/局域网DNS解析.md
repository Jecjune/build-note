建议[[pve系列/远程连接#^c99048|新开个lxc]]容器另外作为DNS服务器以及局域网反向代理，完成容器建立后先进行[[宝塔面板的安装]]再进行以下操作。

这里采用**Dnsmasq**作为DNS解析服务器。

# 简介：
**Dnsmasq** 提供 DNS 缓存和 DHCP 服务功能。作为**域名解析**服务器(DNS)，dnsmasq可以通过缓存 DNS 请求来提高对访问过的网址的连接速度。**作为DHCP 服务器**，dnsmasq 可以用于为局域网电脑分配内网ip地址和提供路由。DNS和DHCP两个功能可以同时或分别单独实现。dnsmasq轻量且易配置，适用于个人用户或少于50台主机的网络。此外它还自带了一个 **PXE** 服务器。

一篇极尽详细的博客：[DNSmasq详细解析及详细配置-腾讯云开发者社区-腾讯云 (tencent.com)](https://cloud.tencent.com/developer/article/1174717)

## 开始

首先关掉系统默认的DNS服务，否则53端口会被占用，如果想保留可以查看[[解决53端口占用问题]]，虽然我感觉没必要，因为后续DNS会由dnsmasq接管
```bash
sudo systemctl stop systemd-resolved
```
```bash
sudo systemctl disable --now systemd-resolved
```

安装
```
apt install dnsmasq
```
设置开机启动：
```
systemctl enable dnsmasq
```

## 解析流程
修改配置之前首先要了解运行流程。

dnsmasq服务在启动时默认会加载`/etc/resolv.conf`文件中定义的DNS服务器。

解析过程中，dnsmasq先去解析hosts文件， 再去解析/etc/dnsmasq.d/下的*.conf文件，并且这些文件的优先级要高于dnsmasq.conf，我们自定义的resolv.dnsmasq.conf中的DNS也被称为上游DNS，这是最后去查询解析的；
如果不想用hosts文件做解析，我们可以在/etc/dnsmasq.conf中加入no-hosts这条语句，这样的话就直接查询上游DNS了，如果我们不想做上游查询，就是不想做正常的解析，我们可以加入no-reslov这条语句。

hosts文件可以用 cat /etc/hosts 查看，/etc/dnsmasq.d下默认没有文件，所以一般来说直接当啥都没，去修改 dnsmasq.conf 就完了。

## 编辑配置文件
编辑 dnsmasq 的配置文件 /etc/dnsmasq.conf 。这个文件包含大量的选项注释，具体解析请看[[Dnsmasq配置文件示例]]

**dnsmasq经常修改的比较重要参数说明**

#resolv-file
定义dnsmasq从哪里获取上游DNS服务器的地址， 默认从/etc/resolv.conf获取。
#strict-order
表示严格按照resolv-file文件中的顺序从上到下进行DNS解析，直到第一个解析成功为止。
#listen-address
定义dnsmasq监听的地址，默认是监控本机的所有网卡上。
#address
启用泛域名解析，即自定义解析a记录，例如：address=/long.com/192.168.115.10 访问long.com时的所有域名都会被解析成192.168.115.10
#bogus-nxdomain
对于任何被解析到此 IP 的域名，将响应 NXDOMAIN 使其解析失效，可以多次指定 通常用于对于访问不存在的域名，禁止其跳转到运营商的广告站点|
#server
指定使用哪个DNS服务器进行解析，对于不同的网站可以使用不同的域名对应解析。 例如：server=/google.com/8.8.8.8    表示对于google的服务，使用谷歌的DNS解析。


## 开始修改Dnsmasq配置文件dnsmasq.conf

增加以下内容：
```
nano /etc/dnsmasq.conf
```

```
strict-order

#设置最大缓存数
cache-size=1000

#局域网域名解析
address=/www.dawalkers.top/192.168.1.202
address=/nas.dawalkers.top/192.168.1.202
address=/gitlab.dawalkers.top/192.168.1.202

#指定部分常用网站解析地址
server=/cn/114.114.114.114
server=/taobao.com/114.114.114.114
server=/taobaocdn.com/114.114.114.114
server=/google.com/8.8.8.8

#屏蔽网页广告用
address=/ad.youku.com/127.0.0.1
address=/ad.iqiyi.com/127.0.0.1
```


修改结束后可以检查语法是否正确：
```php
dnsmasq -test
```
重启dnsmasq：
```php
sudo systemctl restart dnsmasq
```


查看端口占用情况
```bash
sudo netstat -nultp
```

查询dns解析是否设置成功：
```php
nslookup <域名>
```



***注意事项：***
1. listen-address=127.0.0.1   可以用来指定监听的ip地址，防止非法访问，但是我在用的时候，加上之后连目标地址也无法访问了，不知道是哪里的问题，慎用。
2. 推荐的教程中在/etc/dnsmasq.d/新建文件，并在dnsmasq.conf以引用的形式添加hosts和config，在使用的过程中我遇到了语法错误，无法启用服务的情况，删除了才成功，如果非要用的话建议看一下/etc/dnsmasq.d/中的readme和官方文档，总之慎用。
3. 局域网内的域名解析虽然可以通过设置主DNS和辅DNS的方式解决同名问题，但是会出现curl成功了但是仍然在浏览器中无法访问的情况，有可能是因为某种安全策略导致的，毕竟这种操作已经和DNS劫持差不多了，所以建议不要搞公网和局域网都相同的域名解析。



接下来可以继续完成[[局域网反向代理]]，实现网址自动转入局域网服务






***








# bind9的失败尝试，不知道为什么被拒绝访问或者无法提示递归不可用，而且不太稳定，算了，弃用了


采用pve下lxc容器运行dns服务器，本文转载自：[Docker搭建DNS服务器 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/615495383)

先[[pve系列/远程连接#^bb4dff|下载lxc]]并安装docker

创建配置文件存储文件：
```bash
mkdir -p /opt/docker/dns-server
```

### **创建容器运行dns服务器**

```bash
docker run --name dns -d --restart=always --publish 53:53/tcp --publish 53:53/udp --publish 10000:10000/tcp --volume /opt/docker/dns-server:/data sameersbn/bind:latest
```

### **参数说明**

- `-p 53:53/udp` 绑定容器53端口到宿主机的`53`端口，`DNS`默认端口;
- `-p 10000:10000` 图形化界面管理器端口;
- `-volume /dns-server:/data` 挂载本地目录作为`dns`配置存储


如果建立不成功,查看日志和输出,如果是53端口占用,请看[[解决53端口占用问题]]，然后重新建立即可。

如果重新建立时报错说已经docker编号已经被占用，使用即可
```
docker start dns
```

### **访问DNS服务器**

- 访问地址 `https://ip:10000`
- 默认账户和密码`root/password`

![[局域网DNS1.png]]
更改语言，选择简体中文-UTF8，记得刷新网页:
![[局域网DNS2.png]]
修改密码root密码：
![[局域网DNS9.png]]

设定解析默认值
![[局域网DNS3.png]]

创建新的主区域-正向映射
![[局域网DNS4.png]]

![[局域网DNS5.png]]
域名写要解析的域名，如：dawalker.top
email随便写写就好
主服务器指向自己就可以了(localhost)，因为只做本地解析
然后为新建的主区域添加ip映射
![[局域网DNS6.png]]
![[局域网DNS7.png]]
名称就是子级域名，地址就是ip地址

### **重启容器**
```bash
docker restart dns
```

# 客户端

修改路由器DNS，指向DNS服务器地址，让DHCP用DNS的解析

对于静态ip的电脑，手动设定DNS地址，重启systemd-resolved服务
```bash
systemctl restart systemd-resolved

systemctl enable systemd-resolved
```

win10下查询dns解析是否设置成功：
```
nslookup <域名>
```