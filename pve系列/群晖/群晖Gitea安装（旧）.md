# 第一步：安装docker并完成配置
1、打开nas的套件中心，安装docker。
2、由于docker默认安装地址是指向国外网址的，所以需要进行特别的配置。
简单方法：注册表镜像URL填写国内官网：https://registry.docker-cn.com
但是这个经常会下载失败，有点看脸。
稳定的方法是，利用阿里的国内加速镜像源，教程地址：https://help.aliyun.com/zh/acr/user-guide/accelerate-the-pulls-of-docker-official-images
注意，该加速器地址需要注册为阿里的用户才会获得。方法：在阿里注册后在容器镜像服务中找到镜像工具，然后找到镜像加速器，复制其中的网址即可。
3、如果还是无法访问注册表，那么就需要使用命令行的方式进行后续操作。后面就是使用命令行的方式进行配置。
4、电脑安装putty进行ssh连接到群晖（下面有如何链接的方法）
5、在命令行中使用 
```php
docker search gitea
```
 指令搜索镜像文件，然后使用 
```php
docker pull <镜像名> <标签名>
```   
安装 gitea/gitea ，标签可以不加，默认是最新版本。或者直接网页进入docker的官方仓库，找到需要的镜像，复制对应的拉取指令来进行安装。（官网好像进不去了）

如果下载速度很慢，或者下载失败，可以尝试使用[Docker Proxy 镜像加速下载](https://dockerproxy.com/)，进入网站之后，在第一步中贴上需要下载的镜像名字，然后第二步就会生成命令，直接复制过去群晖中运行即可

6、下载完成后，返回群晖网页的操作界面，就可以在docker的映像中找到刚刚拉取的gitea了
# 第二步，安装mysql数据库服务
同样和gitea一样安装即可。
### 配置mysql。
在file station中docker文件夹下新建一个mysql文件夹，并在属性设置权限，群组设为everyone，权限全部选上，确定。然后把应用到子文件夹也选上。然后到docker的映像中配置sql。本地端口设置为33060，容器端口设置为3306。（其实本地端口找一个不冲突的就可以了，但是容器端口号必须为3306，这个是mysql映射出来给用户登录的唯一端口）
高级设置中，将存储位置设置为之前新建的那个文件夹中，装载路径就是mysql在容器中的地址，可以设置为：/var/lib/mysql 。记得把只读取消勾选，否则无法启动
环境变量添加root账号的密码（变量名设置为：MYSQL_ROOT_PASSWORD   值就是要设置的密码）
### 启动mysql。
在容器中双击新增的容器，然后在terminal页点击新增-通过命令启动。填入：/bin/bash
然后就会在左边栏中增加一个bash，选中后输入指令：mysql -uroot -p<密码>  就可以进入mysql数据库中了。
然后修改root用户远程连接权限：grant all privileges on *.* to ‘root’@’%’ ;
在通过sql查验是否设置成功：select Host,User,plugin from mysql.user;（如果root的host是%就表示任何root账号的ip地址都可以连接，localhost表示只有本地ip可以连接）
### 安装mysql数据库可视化工具：adminer    
教程：[强大的网页数据库管理工具Adminer-CSDN博客](https://blog.csdn.net/wbsu2004/article/details/121949994)
按上面的步骤在docker安装adminer，然后配置文件夹和端口，配置完成后直接输入群晖ip地址加对应的本地端口号即可访问adminer
登陆后，新建数据库，编码选择： utf8mb4_general_ci

# 第三步在docker中安装gitea，并完成容器配置。
设置端口映射，容器端口号为3000，本地端口号自选。
添加文件夹docker/gitea    装载路径为 /data/gitea
添加文件夹docker/gitea/root  装载路径为 /data/git/repositories               注意：记得取消只读勾选！！
链接选项，将mysql添加进来，别名随便取，我设为my-gitea
 //环境选项，USER    git
	//	GITEA_CUSTOM   /data/gitea
	//	PATH /usr/local/sbin:/usr/local/bin:/git     （环境选项别写，写了就不行，这里只做记录）
运行容器，并打开刚刚设置的端口地址（群晖ip地址+映射的本地端口地址）
第一次进入git需要配置
	数据库主机：这里写mysql的地址（ip+本地端口号）
	用户名：root
	数据库名称：要和之前建立给gitea的那个名称一致
	服务器域名：写群晖的ip地址
	基础URL：http://<群晖地址ip>:<getea端口>/
        可选设置：管理员账号建议先建立

后续。后面如果通过花生壳等网站进行了域名映射，只要将新域名在/docker/gitea/config/app.ini   配置文件中替换掉群晖的ip地址，就能够让gitea的仓库地址改变了

## 常见报错：
1、Are you trying to connect to a TLS-enabled daemon without TLS?
			--没有以root权限运行指令
3、使用docker运行gitea时，报错unable to exec s6-supervise: no such file or directory
	不要填写环境变量，重新创建容器



$$常用docker源$$
Dockerhub仓库1：https://hub.docker.com/
Dockerhub仓库2：https://registry.hub.docker.com
镜像代理加速网站：https://dockerproxy.com/
镜像加速器地址：https://gist.github.com/y0ngb1n/7e8f16af3242c7815e7ca2f0833d3ea6
		https://registry.hub.docker.com/