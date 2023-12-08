
证书由颁发机构（CA）签发，CA提供了一种获得和安装免费[TLS/SSL证书](https://link.juejin.cn/?target=https%3A%2F%2Fwww.digitalocean.com%2Fcommunity%2Ftutorials%2Fopenssl-essentials-working-with-ssl-certificates-private-keys-and-csrs "https://www.digitalocean.com/community/tutorials/openssl-essentials-working-with-ssl-certificates-private-keys-and-csrs")的简单方法，从而在网络服务器上实现加密的HTTPS。



# 准备活动
1.一台具有公网ip的服务器/云主机
2.购买了至少一个域名

这里环境为阿里云的轻量级服务器和namesilo的域名服务.
# 添加域名解析
到域名购买商提供的dns服务中添加A解析记录(namesilo的dns服务是免费提供的):
![[SSL申请.png]]

A记录指的是ipv4的地址,AAAA指的是ipv6的地址
hostname是子级域名,随便写就好.
ipv4 address就是公网ip地址

提交后等待解析成功,此时已经可以通过域名访问到公网ip了

# 添加站点信息
这里采用宝塔面板作为管理工具
首先添加站点信息,域名是必要的,其他等创建完成后再继续即可:
![[SSL申请3.png]]




# 进行dns验证

这里选择阿里云的免费ssl服务，首先购买免费证书，然后点击创建证书

![[SSL申请7.png]]

域名验证方式选择手动验证，CSR为系统生成
![[SSL申请8.png]]

然后会生成表单，选择查看进度
![[SSL申请9.png]]

这里会提出验证要求
![[SSL申请10.png]]

回到namesilo的dns页，添加记录，然后等待解析完成，大概五分钟。
![[SSL申请11.png]]

然后回到阿里云点击验证。
![[SSL申请13.png]]

验证成功会自动进入证书签发环节，耐心等待即可。签发成功的样子：
![[SSL申请14.png]]

此时可以回到dns服务器，删掉用于认证的txt信息了。
看自己宝塔面板使用的工具是什么，点击下载对应的证书，一般是nginx
![[SSL申请15.png]]
下载完成后解压，里面会有key和pem两个文件，粘贴到宝塔中即可。
![[SSL申请16.png]]

保存并启用后，重新打开站点信息，勾选强制HTTPS。
![[SSL申请17.png]]


至此，完成配置证书申请。
接下来可以通过[[基于宝塔面板的反向代理]]隐藏端口号，进行直接的服务-网址映射。

