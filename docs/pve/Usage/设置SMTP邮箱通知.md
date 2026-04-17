
## QQ邮箱通知设置
1. 数据中心，选项，设置好“来自…邮件”。
这个是邮箱中显示发件人邮箱地址的设置。
2. 数据中心，权限，用户，设置好root账户的邮箱。
![邮箱设置1.png](../picture/邮箱设置1.png)

3. 安装libsasl2-modules。记得换源、apt update。
```php
apt install libsasl2-modules -y
```

 4. 重新加载postfix
```php
postfix reload
```

5. 新建SMTP账户： ![邮箱设置2.png](../picture/邮箱设置2.png)
下面是qq邮箱的设置示例：
![邮箱设置3.png](../picture/邮箱设置3.png)


6. 测试邮箱是否正常:
![邮箱通知2.png](../picture/邮箱通知2.png)

7. 设定通知匹配器
将通知目标设置为刚刚新建的smtp账号
![邮箱设置4.png](../picture/邮箱设置4.png)