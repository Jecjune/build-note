# 1.项目中管理项目成员
在项目页找到需要管理的项目
![[Pasted image 20231215113638.png]]

进入后，左边栏就会改变为管理项，在里面管理即可。
![[Pasted image 20231215113721.png]]


# 2.浏览他人的项目
点击狐狸头回到主页，右边有个`浏览项目`，点击后就能看到公开但是自己没有加入的项目了。如果想要别人能直接看到自己的项目，可以邀请他人加入为项目成员，或者直接将群组邀请到项目中去。
![[1702611506283.png]]

# 3.上传代码流程
1. 首先在gitlab上面新建项目：
![[2024-01-14_22-56.png]]
2. `是否使用自述文件初始化`选项建议不要开启，因为非空白的项目上传时需要解决合并问题，不懂什么意思的直接不要开启就好了。
![[2024-01-14_22-57.png]]
3. 建立工程后，点击克隆，然后点击使用HTTP克隆，复制链接下来![[2024-01-14_23-02.png]]
4. 在代码的主目录中打开命令行
windows下：右键，`git bash here`
Linux下：右键，在此打开终端

5. 初始化仓库
输入指令：
```php
git init
```
6. 将主目录下的文件加入检查列表：
```php
git add .
```
7. 添加描述：
```php
git commit -m "注释内容"
```
8. 添加远端仓库地址：
```php
git remote add <url> <remote>
``` 
*url*:一个给远端地址备注的名称，随便取
*remote*:新建完工程后点击克隆复制到的那个链接
<>括号在指令中是没有的，它是一种表示方法，仅表示这是一个要输入的参数而已
9. 设置当前的远端地址：
```php
git push --set-upstream <url> <branch>
```
*branch*：远端的目标分支


至此，新工程的建立和上传已经结束。
