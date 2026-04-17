# fatab文件概述

在linux系统中，`/etc/fstab`文件用于描述当前系统中挂载的各种设备对应的文件系统信息。系统会在开机时读取该文件内容，并根据配置信息挂载磁盘，当文件信息错误时可能会导致系统无法完成挂载任务，无法开机。

## 基本语句：
```php
/dev/sdg1 /media/backup jfs defaults 0 2
```

- `/dev/sdg1` :第一个参数是物理地址，可以是盘符也可以指定uuid或者设备名（盘符要注意有盘符漂移问题）
- `/media/backup`：第二个参数是要挂载到的路径，随便填
- `jfs`：文件格式，一般是ext4，看需求
- `defaults`：这个参数是挂载选项，可以指定多个参数
- `0`： dump工具选项，决定是否备份。0表示忽略，1表示进行备份
- `2`：最后一个参数，fsck选项，决定是否需要检查文件系统，以及检查顺序。0：不检查。1：最高优先权，根目录应该设置为1。2：次要优先权，一般设置为这个


## 挂载选项
注意，部分选项是特定文件系统才有的。
```
一些比较常用的参数有：
auto - 在启动时或键入了 mount -a 命令时自动挂载。
noauto - 只在你的命令下被挂载。
exec - 允许执行此分区的二进制文件。
noexec - 不允许执行此文件系统上的二进制文件。
ro - 以只读模式挂载文件系统。
rw - 以读写模式挂载文件系统。
user - 允许任意用户挂载此文件系统，若无显示定义，隐含启用 noexec, nosuid, nodev 参数。
users - 允许所有 users 组中的用户挂载文件系统.
nouser - 只能被 root 挂载。
owner - 允许设备所有者挂载.
sync - I/O 同步进行。
async - I/O 异步进行。
dev - 解析文件系统上的块特殊设备。
nodev - 不解析文件系统上的块特殊设备。
suid - 允许 suid 操作和设定 sgid 位。这一参数通常用于一些特殊任务，使一般用户运行程序时临时提升权限。
nosuid - 禁止 suid 操作和设定 sgid 位。
noatime - 不更新文件系统上 inode 访问记录，可以提升性能(参见 atime 参数)。
nodiratime - 不更新文件系统上的目录 inode 访问记录，可以提升性能(参见 atime 参数)。
relatime - 实时更新 inode access 记录。只有在记录中的访问时间早于当前访问才会被更新。（与 noatime 相似，但不会打断如 mutt 或其它程序探测文件在上次访问后是否被修改的进程。），可以提升性能(参见 atime 参数)。
flush - vfat 的选项，更频繁的刷新数据，复制对话框或进度条在全部数据都写入后才消失。
defaults - 使用文件系统的默认挂载参数，例如 ext4 的默认参数为:rw, suid, dev, exec, auto, nouser, async.

```

### 使用示例
1. 设置home自动挂载
该设置在/home分区较大的时候，可以让不依赖于/home分区的服务先启动。系统会缓存所有操作，等待/home挂载再进行，从而加速启动。值得注意的是，在未挂载期间，/home文件系统会被识别为autofs，可能会导致一些检查工作跳过，加速效果也因配置而异。
```
/dev/sda1 /home ext4 noauto,x-systemd.automount 0 2
```

2. 设置可选加载和超时时间
有些硬盘中的数据可能没有运行主要服务所依赖的，可以设置为`无报错加载`，这样可以在该设备无法找到时也正常进入系统，对于需要经常插拔的设备有一定的便利。同时，由于默认加载失败时间为60s，所以还可以重设等待时间，避免开机硬等待。
```
/dev/sdg1 /media/backup jfs nofail,x-systemd.device-timeout=1 0 2
```

3. 设置加载依赖
有些外部设备可能需要在某些依赖服务启动后再加载。
```
/share /net/share cifs noauto,x-systemd.automount,x-systemd.requires=network-online.target,x-systemd.device-timeout=10,credentials=/foo/credentials 0 0
```

4. 指定用户
某些设备可能需要指定使用用户
```
/share /share cifs noauto,workgroup=workgroup 0 0
```