# 01镜像的烧录与配置文件的修改

先用diskgenius刷成guid分区fat32格式的，再用rufus4.2p进行刷入即可。
安装过程直接选用图形化安装就好，ip设置要记住，其他的注意一下安装硬盘位置和网卡的选择。安装完成后，可以通过ssh连接22端口进行访问，默认账号为root，密码为安装时设置的，推荐使用Mobaxterm，或者通过浏览器连接8006端口访问控制台UI。

#### 进入系统后,需要修改gurb启动参数方便后续直通:
```php
nano /etc/default/grub
```
找到下面那句,并改为,非常注意的是，***把下面那行空的GRUB_CMDLINE_LINUX_DEFAULT删掉，因为只有最后一次赋值会有效***
```
GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=on iommu=pt pcie_acs_override=downstream"
```
AMD则改成
```
quiet amd_iommu=on iommu=pt pcie_acs_override=downstream
```

***quiet***: 禁止内核在启动时打印大量的启动信息，以减少控制台输出的噪声。
***intel_iommu=on***: 启用 Intel I/O 虚拟化技术，这对于支持 SR-IOV 和 PCI Passthrough 的虚拟化环境非常有用。
***iommu=pt***: 指定 I/O 虚拟化使用页表（PT）映射模式。
***pcie_acs_override=downstream***: 启用 PCIe ACS（Access Control Services）覆盖机制以允许跨 PCIe 设备之间的 DMA 流量。downstream 选项允许将 DMA 流量限制为从 PCIe 根端口向下游设备流动。

最后执行:
```php
update-grub
```
#### 添加硬件驱动黑名单:
```php
nano /etc/modprobe.d/pve-blacklist.conf
```
添加以下内容:
```
blacklist nvidiafb 
blacklist amdgpu 
blacklist i915 
blacklist snd_hda_intel 
options vfio_iommu_type1 allow_unsafe_interrupts=1
```
最后执行:
```php
update-initramfs -u -k all
```

### 加载vfio驱动
```php
nano /etc/modules
```
加入以下内容：
```
vfio
vfio_iommu_type1
vfio_pci
vfio_virqfd
```



***
# 02网络设置
## 1.静态IP设置
建议直接在pve控制面板上设置，方便很多。先查看路由器中LAN口的ip设置和子网掩码，然后将pve设置在同一网段即可。必须要注意的是，静态IP不要设置在DHCP服务器的网段中，否则可能出现IP冲突，导致路由器无法成功分配IP。IP正确设置后如果无法ping通域名，可能是DNS的配置问题，查询路由器的DNS服务器并对着配置即可。

DNS服务器配置：
```php
nano /etc/network/interfaces
```
当存在远程连接时，可能IP的修改无法立即生效，可以通过重启网络服务（有时不行）或重启机器的方式加载新配置。
重启网络服务：
```php
service networking restart
```

## 2.动态IP设置（静态设置成功就不用管这个了）
如果网络配置错误，那么pve将无法ping通网络，可以通过设置为DHCP模式自动获取ip地址。
```php
nano /etc/network/interfaces
```

然后找到正在使用的网卡，设置为以下形式：
```json
auto vmbr0
iface vmbr0 inet dhcp
        bridge-ports enp4s0
        bridge-stp off
        bridge-fd 0
        bridge-vlan-aware yes
        bridge-vids 2-4094
```

**值得注意的是：当设置为动态获取之后，每次开机ip都可能不一致，必须要获取到当前ip才能连接上pve。方法如下：登录路由器查看或者登录到物理主机使用 hostname -I 查看。

## 3.修改开机界面的URL内容
文件位置为：
```php
nano /etc/issue
```

## 4.修改网卡驱动（可选）
建议先修改完软件源再过来.
查看当前物理网卡：
```php
ip link show
```
![[网卡驱动1.png]]
查看网卡驱动信息:
```php
ethtool -i enp4s0
```
![[网卡驱动2.png]]
查看网卡厂商信息:
```php
lspci | grep -i ethernet
```
![[网卡驱动3.png]]

主要对比厂商信息和驱动信息，确定驱动是否正确。基于上面的信息，我们知道我们当前的网卡型号是`RTL8111/8168/8411`，而我们实际使用的驱动是`r8169`，那么在这种情况下就会出现网卡与驱动不匹配，进而导致网络时断时续等问题。
当前驱动信息就是不正确的，需要下载重安装。
瑞星驱动下载官网：[Realtek PCIe FE / GBE / 2.5G / 5G Ethernet Family Controller Software - 瑞昱半导体](https://www.realtek.com/zh/component/zoo/category/network-interface-controllers-10-100-1000m-gigabit-ethernet-pci-express-software)
![[网卡驱动4.png]]
上传到pve中，解压：
```php
tar -jxvf r8168-8.052.01.tar.bz2
```
**由于pve裁剪了许多功能，包括基本的编译工具，因此不要直接执行安装脚本，否则网卡驱动被卸载后将无法链接网络，先手动编译看是否能通过以检查环境。**
默认没有make和gcc，首先需要安装这两个：
```php
apt-get update && apt install make gcc build-essential
```
以及pve的相关内核文件
```php
apt install pve-headers-6.5.11-4-pve
```
注意，后面的呃版本号要对应自己的pve系统，使用uname -r 可查看

然后进入src文件夹，进行编译：
```php
make all && make install
```

出现警告Warning: modules_install: missing 'System.map' file. Skipping depmod.
是因为缺少文件导致,解决方法如下:
```php
ln -s /boot/System.map-6.5.11-4-pve /lib/modules/6.5.11-4-pve/build/System.map
```
当编译没有存在问题的时候,就可以运行脚本进行安装了
运行脚本时出现Permission denied是因为没有可执行权限,添加上即可:
```php
chmod +x autorun.sh
```
指令安装指令后,网卡会被卸载并重装,此时网络会断开,等待一两分钟后重新插拔网线刷新物理网卡即可重新链接网络.(不行的话请尝试重启)
安装成功后效果:
![[网卡驱动5.png]]


***

# 03系统源的更换与软件更新

默认是企业订阅版，如果不做修改，在使用 pveceph init 进行 ceph 初始化安装的时候会将整个环境破坏，切记！
**【重要】将/etc/apt/sources.list.d/pve-enterprise.list 文件删除或注释：
```bash
rm -rf /etc/apt/sources.list.d/pve-enterprise.list
```
### 1.Proxmox软件源更换

pve软件源配置文件：
```copy
nano /etc/apt/sources.list.d/pve-no-subscription.list
```
中科大源（选了这个）：
```php
echo "deb https://mirrors.ustc.edu.cn/proxmox/debian/pve bookworm pve-no-subscription" > /etc/apt/sources.list.d/pve-no-subscription.list
```

清华源：
```php
# 清华Tuna源
echo "deb https://mirrors.tuna.tsinghua.edu.cn/proxmox/debian bookworm pve-no-subscription" > /etc/apt/sources.list.d/pve-no-subscription.list
```
### 2.Debian系统源更换

配置文件为：
```copy
nano /etc/apt/sources.list
```
阿里Debian源（选了这个）：
```php
sed -i.bak "s#ftp.debian.org/debian#mirrors.aliyun.com/debian#g" /etc/apt/sources.list     #阿里Debian源
```
```php
sed -i "s#security.debian.org#mirrors.aliyun.com/debian-security#g" /etc/apt/sources.list     #阿里Debian源
```
```bash
apt update && apt-get install -y apt-transport-https ca-certificates  --fix-missing
```
华为Debian源：
```php
sed -i.bak "s#http://ftp.debian.org#https://repo.huaweicloud.com#g" /etc/apt/sources.list     #华为Debian源
```
```php
sed -i "s#http://security.debian.org#https://repo.huaweicloud.com/debian-security#g" /etc/apt/sources.list     #华为Debian源
```
```bash
apt update && apt-get install -y apt-transport-https ca-certificates  --fix-missing
```
### 3.LXC（容器）仓库源更换

配置文件为：
```copy
nano /usr/share/perl5/PVE/APLInfo.pm
```
中科大源（选了这个）：
```php
sed -i.bak "s#http://download.proxmox.com/images#https://mirrors.ustc.edu.cn/proxmox/images#g" /usr/share/perl5/PVE/APLInfo.pm  
```
```
wget -O /var/lib/pve-manager/apl-info/mirrors.ustc.edu.cn https://mirrors.ustc.edu.cn/proxmox/images/aplinfo-pve-7.dat
```
```bash
systemctl restart pvedaemon
```
南大NJU源：
```csharp
sed -i.bak "s#http://download.proxmox.com/images#https://mirrors.nju.edu.cn/proxmox/images#g" /usr/share/perl5/PVE/APLInfo.pm  
```
```
wget -O /var/lib/pve-manager/apl-info/mirrors.nju.edu.cn https://mirrors.nju.edu.cn/proxmox/images/aplinfo-pve-7.dat
```
```bash
systemctl restart pvedaemon
```
### 4.CEPH源更换
```php
echo "deb https://mirrors.ustc.edu.cn/proxmox/debian/ceph-quincy bookworm no-subscription" > /etc/apt/sources.list.d/ceph.list     #中科大源
```
```php
sed -i.bak "s#http://download.proxmox.com/debian#https://mirrors.ustc.edu.cn/proxmox/debian#g" /usr/share/perl5/PVE/CLI/pveceph.pm     #中科大源
```

### 5.删除订阅弹窗
```bash
sed -Ezi.bak "s/(Ext.Msg.show\(\{\s+title: gettext\('No valid sub)/void\(\{ \/\/\1/g" /usr/share/javascript/proxmox-widget-toolkit/proxmoxlib.js && systemctl restart pveproxy.service
# 执行完成后，浏览器Ctrl+F5强制刷新缓存
```
### 6.最后执行一次软件更新
```php
apt update && apt dist-upgrade     #更新软件，可不执行
```

***

# 04存储空间的调整

默认情况下,PVE安装好之后会在硬盘上创建两个区域,一个命名为local,另一个上local-lvm,前者支持文件目录结构,简而言之就是可以看到具体的文件夹和文件,而后者local-lvm是不支持文件目录结构的,通常虚拟磁盘就存储在这个部分,存储类别为 lvm-thin。

前言：
该步骤适合以下两种需求的人进行，否则可以直接跳过：
1.需要调整local存储池大小的：觉得local不够大或者觉得太大（建议保留30~40G）。
2.有多开需求的（一下创建50个云桌面之类）。

关于lvm-thin：LVM的精简逻辑卷，这可以创建大于可用盘区的逻辑卷。精简池可以在需要时进行动态扩展，存储管理员可以过量使用物理存储，以节省成本地分配存储空间。在标准的逻辑卷中磁盘空间在创建时就会占用卷组的空间，但是在精简卷中只有在写入时才会占用存储池"thin pool LV"中的空间，且支持数据丢弃（已经占用过又释放的空间可以重新投入使用）。不过默认的local-lvm由于没有文件目录的缓存，IO效率不足且需要更多运算量，所以在多开和大容量调用时，会出现执行时间更长的情况。

查看当前内存使用情况和各盘的文件系统：
```php
df -hT
```
对于上述问题，以下提出两种解决方案：
### 1.合并local和local-lvm

通过以下步骤将local-lvm进行删除并重新分配到pve的可见目录（root）下：
```php
lvremove pve/data && lvextend -l +100%FREE -r pve/root
```
但是如果数据盘和系统盘放在一起，当容量不够时会挤占系统盘的位置，最后可能导致系统崩溃无法启动。

### 2.调整local的大小并重建local-lvm

#### 2.1 local的调整
建议local保留大小为30~40G，如果占用没有太离谱，不建议进行调整。
具体操作方法：[[如何调整LVM相关PV、VG和LV大小#^ebd0a9|LVM相关-LV大小的调整]]

*local是作为逻辑卷存在的，是不能直接使用lvreduce做删减动作的，不然就会崩掉。然后这里记录一下直接使用lvreduce做删减动作后的抢救方法：
1.重启并进入紧急抢救模式，在lvm中是支持卷操作的，所以可以通过命令行进入lvm进行恢复操作。
2.修改/etc/lvm/lvm.conf文件，将locking_type=4改为locking_type=1
3.直接用扩容命令lvextend -l +100%FREE /dev/mapper/centos-root恢复逻辑卷大小
4.exit退出lvm操作，再重新挂载根目录，这里感觉不用挂载根目录直接重启系统也行。

**最后补充，遇到一直无法进入emergency mode，每次重启都是会卡住，界面上没有出现任何的命令，最后发现内核配置删掉串口连接，即去掉console=ttyS0，ctrl + X就进入emergency mode了。

#### 2.2 local-lvm的重建

##### 第一步，从Proxmox Web界面，删除 local-lvm
数据中心->存储中，选择 local-lvm，然后点击删除。
##### 第二步，通过命令删除 lvm，新建lvm，并创建文件系统。
卸载并删除 lvm-thin
```php
umount /dev/pve/data && lvremove /dev/pve/data
```
检查磁盘剩余空间
```php
vgdisplay pve | grep Free
```
##### 第三步，创建新的lvm并挂载。
使用lvm创建新的lvm-data存储
```php
lvcreate -l <剩余空间> -n data pve
```
格式化为ext4文件系统
```php
mkfs.ext4 /dev/pve/data
```
新建挂载路径
```php
mkdir /mnt/data
```
完成挂载
```php
mount /dev/pve/data /mnt/data
```
##### 第四步，配置fstab,确保重启时能够挂载文件系统。
```php
nano /etc/fstab
```
设置为:
```txt
/dev/pve/data /mnt/data ext4 defaults 0 0
```
##### 第五步，在Promox Web页面将新的lvm注册为存储。
选择 DataCenter->存储->添加， ID填写 data， 目录填写 /mnt/data。


***





***至此,基本安装步骤和软件已经安装完毕***