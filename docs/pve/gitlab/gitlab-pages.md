使用 GitLab Pages，可以直接从 GitLab 的存储库中发布静态网站。

## 开启pages
默认 gitlab 的 pages 是关闭的，我们需要开启。由于我们的gilab是由群晖的docker搭建的，所以我们需要转到群晖的docker详情页，新建一个bash进行gitlab的配置。
编辑 gitlab.rb 文件：
```
vi /etc/gitlab/gitlab.rb
```
加入下面的代码，保存：
```
pages_external_url "http://pages.example.com/"
gitlab_pages['enable'] = true

 gitlab_pages['artifacts_server'] = true
 gitlab_pages['artifacts_server_url'] = nil # Defaults to external_url + '/api/>
 gitlab_pages['artifacts_server_timeout'] = 10

```
`pages_external_url`就是`gitlab`的域名，该域名不能和`gitlab`仓库域名一致，否则将无法使用。因为`gitlab`和`gitlab-pages`是共用80端口的，只不过通过反向代理将指向`gitlab-pages`的流量转发到了本地（127.0.1）的8090端口实现`gitlab-pages`的访问，如果域名一致，将无法实现代理。

`artifacts_server`是管理`gitlab-runner`运行产物的模块。

重新加载配置：
```
gitlab-ctl reconfigure
```
确认没有报错后，回到gitlab网页，等待服务重新开启后，就可以在项目的`部署`选项中找到`pages`的设置。


## 修改云服务端相关配置

1. 一般gitlab_pages提供的静态网页会添加用户名作为子级域名，如`http://root.dawalker.sbs/build-note/pve/gitlab/gitlab-pages/`中的`root`就是用户名。所以需要先在你的dns服务器上做泛域名的解析，将所有`dawalker.sbs`下的子域名都解析为你的云服务器ip地址。这里以域名`dawalker.top`为例描述以下进行泛域名解析流程：进入你的DNS服务器后台->添加A记录->将子域名设置为`*.dawalker.top`
2. 如果你用的是宝塔面板，还需要在云服务器中的宝塔面板中增加泛域名网站的记录：[泛域名添加站点](../网络管理/局域网反向代理.md#da77de)（因为后续需要进行泛域名的反向代理）。
3. 解析完并添加站点后，还需要进行一个[泛域名的反向代理](../网络管理/局域网反向代理.md#da88de)，将流量重新指向正确的端口（`gitlab`的80端口，如果进行了frp就看frp后的是什么端口）


## 编写.gitlab-ci.yml

为了让`gitlab-runner`执行自动部署，我们需要配置流水线任务，使gitlab完成静态网页的持续部署，`.gitlab-ci.yml`就是一个指示`gitlab-runner`工作流程的配置文件。
这是一个gitlab自动部署的例程，可以拉取下来参考配置：
```
https://gitlab.com/pages/hexo.git
```


## 查错需要用到的
gitlab的nginx配置地址：
```
cd /var/opt/gitlab/nginx/conf
```

查看gitlab pages日志：
```
gitlab-ctl tail gitlab-pages
```
