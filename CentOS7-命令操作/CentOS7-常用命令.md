# 常用安装命令



## 下载源

### 添加epel源

```shell
# 添加第三方源
yum install epel-release -y
```

### yum换源

```shell
yum install -y wget

# 备份当前yum源（可选）
cd /etc/yum.repos.d/
cp /CentOS-Base.repo /CentOS-Base-repo.bak

# 下载阿里yum源repo文件
wget http://mirrors.aliyun.com/repo/Centos-7.repo

# 清理默认缓存包
yum clean all

# 把下载下来的阿里云repo文件设置成为默认源
mv Centos-7.repo CentOS-Base.repo

# 生成阿里云yum源缓存并更新yum源
yum makecache
```

### pip换源

```shell
[root@node01 ~]# mkdir ~/.pip
[root@node01 ~]# vim ~/.pip/pip.conf 
[root@node01 ~]# cat ~/.pip/pip.conf 
[global]
timeout=6000
index-url = https://mirrors.aliyun.com/pypi/simple/
[install]
trusted-host = mirrors.aliyun.com
```

### docker换源

```shell
vim /etc/docker/daemon.json
# 任选一个
{ "registry-mirrors": ["https://9cpn8tt6.mirror.aliyuncs.com"] }
{ "registry-mirrors": ["http://hub-mirror.c.163.com"] }
```

### go换地址

```shell
# 换成国内下载地址
go env -w GO111MODULE=on
go env -w GOPROXY=https://goproxy.io,direct
```



## 常用软件

### netstat

```shell
# 网络常用工具
yum install net-tools
```

### vim

```shell
# 文本编辑常用工具
yum install vim-*
```

### lrzsz

```shell
# 服务器上传拉取常用工具
yum install lrzsz
```

### storcli

```shell
# 首先下载 storcli rpm 包
# yum本地安装
yum localinstall storcli-XXXXXXXXX.rpm -y --disablerepo=*
# 建立软链接
ln -s /opt/MegaRAID/storcli/storcli64 /usr/sbin/storcli
```





# 常用操作命令



## 系统层面

### 查看IPMI

```shell
# 查看是否加载ipmi模块
lsmod | grep ipmi
# 查看ipmi地址
ipmitool lan print 1
```

### 清除缓存

```shell
echo 3 > /proc/sys/vm/drop_caches
```

### 插拔盘

```shell
# 确认盘位置，以/dev/sdd为例
[root@node1 ~]# lsscsi
[14:0:0:0]   disk    ATA      Micron_5200_MTFD H027  /dev/sda 
[14:0:1:0]   disk    ATA      Micron_5200_MTFD H027  /dev/sdb 
[14:0:2:0]   disk    ATA      HGST HUS726T4TAL W41G  /dev/sdc 
[14:0:3:0]   disk    ATA      HGST HUS726T4TAL W41G  /dev/sdd 
[14:0:4:0]   disk    ATA      HGST HUS726T4TAL W41G  /dev/sde 
[14:0:5:0]   disk    ATA      HGST HUS726T4TAL W41G  /dev/sdf 
[14:0:6:0]   disk    ATA      HGST HUS726T4TAL W41G  /dev/sdg 
[14:0:7:0]   disk    ATA      HGST HUS726T4TAL W41G  /dev/sdh 
[14:0:8:0]   disk    ATA      HGST HUS726T4TAL W41G  /dev/sdi 
[14:0:9:0]   disk    ATA      HGST HUS726T4TAL W41G  /dev/sdj 
[14:0:10:0]  disk    ATA      HGST HUS726T4TAL W41G  /dev/sdk 
[14:0:11:0]  disk    ATA      HGST HUS726T4TAL W41G  /dev/sdl 
[14:0:12:0]  disk    ATA      HGST HUS726T4TAL W41G  /dev/sdm 
[14:0:13:0]  disk    ATA      HGST HUS726T4TAL W41G  /dev/sdn 
[14:0:14:0]  enclosu MSCC     SXP 68x12G       RevB  -        
[14:0:15:0]  enclosu UN       Smart Adapter    2.30  -        
[14:1:0:0]   disk    UN       LOGICAL VOLUME   2.30  /dev/sdo 
[14:2:0:0]   storage UN       P460-M4          2.30  -        
# 拔盘
echo 1 > /sys/block/sdd/device/delete
# 插盘
echo " 0 3 0 " > /sys/class/scsi_host/host14/scan
```

### 磁盘检测

```shell
# 磁盘io检测
iostat -tkx /dev/nvme[0-1]n1 1
# -c 仅显示CPU统计信息.与-d选项互斥.
# -d 仅显示磁盘统计信息.与-c选项互斥.
# -k 以KM为单位显示每秒的磁盘请求数,默认单位块.
# -x 显示详细信息

iostat -tkx /dev/sd[c-n] 1

rrqm/s：每秒进行 merge 的读操作数目（当系统调用需要读取数据的时候，VFS将请求发到各个FS，如果FS发现不同的读取请求读取的是相同Block的数据，FS会将这个请求合并Merge）
rsec/s：每秒读取的扇区数。
avgrq-sz 平均每次设备I/O操作的数据大小 (扇区)
avgqu-sz 平均I/O队列长度，越短越好。    
await：平均每次设备I/O操作的等待时间 (毫秒)，即IO的响应时间，包括队列时间和服务时间。
svctm 平均每次设备I/O操作的服务时间 (毫秒)。如果svctm的值与await很接近，表示磁盘性能很好。
%util： 在统计时间内所有处理IO时间，除以总共统计时间。一般地，如果该参数是100%表示设备已经接近满负荷运行了（当然如果是多磁盘，即使%util是100%，因为磁盘的并发能力，所以磁盘使用未必就到了瓶颈）。
```

### 网卡

```shell
# 查看网卡信息
nmcli con show

# /etc/sysconfig/network-scripts/ifcfg-*
# Linux 默认配置网卡的信息
TYPE=Ethernet                               网卡类型:以太网
PROXY_METHOD=none                           代理方式:关闭状态
BROWSER_ONLY=no                             只是浏览器(yes|no)
BOOTPROTO=static                            *设置网卡获得ip地址的方式(static|dhcp|none|bootp)
DEFROUTE=yes                                设置为默认路由(yes|no)
IPV4_FAILURE_FATAL=no                       是否开启IPV4致命错误检测(yes|no)
IPV6INIT=yes                                IPV6是否自动初始化
IPV6_AUTOCONF=yes                           IPV6是否自动配置
IPV6_DEFROUTE=yes                           IPV6是否可以为默认路由
IPV6_FAILURE_FATAL=no                       是不开启IPV6致命错误检测
IPV6_ADDR_GEN_MODE=stable-privacy           IPV6地址生成模型
NAME=eth0                                   网卡物理设备名称
UUID=6e89ea13-f919-4096-ad67-cfc24a79a7e7   UUID识别码
DEVICE=eth0                                 网卡设备名称
ONBOOT=no                                   *开机自启(yes|no)
IPADDR=192.168.103.203                      *IP地址
NETNASK=255.255.255.0                       *子网掩码,也可使用掩码长度表示(PREFIX=24)
GATEWAY=192.168.103.1                       *网关
DNS1=114.114.114.114                        *首选DNS
DNS2=8.8.8.8                                备用DNS
```



## 软件层面

### tar操作

```shell
# 解压示例
wget http://downloads.es.net/pub/iperf/iperf-3.0.6.tar.gz
tar zxvf iperf-3.0.6.tar.gz
cd iperf-3.0.6
./configure
make
make install

# 压缩dir目录
tar -zcf xxx.tar.gz <dirname>
```

### storcli操作

```shell
# 显示物理盘信息
storcli /call/eall/sall show
storcli /call/vall show
storcli /call/vall show all J

# 用于修复物理盘
storcli /call/eall/sall set good
storcli /call/fall import
```

### rpm操作

```shell
rpm -ivh rpm包名
rpm --force -i rpm包名	# 强制安装
rpm -Uvh rpm包名          # 升级软件
rpm -e rpm包名            # 卸载
rpm -e --nodeps rpm包名   # 强制卸载
rpm -qpi rpm包名		# 查询软件包的详细信息
rpm -qf rpm包名		# 查询某个文件是属于那个rpm包的
rpm -qpl rpm包名		# 查该软件包会向系统里面写入哪些文件
rpm -qa 			 # 列出所有安装过的包
```

### screen操作

```shell
# 新建一个名字为 name 的 session
screen -S name
# 查看存在的工作窗口  
screen  -ls
# 切换到要选择的工作窗口
screen -r  xxxx 
# 挂起screen
ctrl+a+d
# kill 掉 screen 命令
screen -wipe
# 删除 122128 screen
screen -X -S 122128 quit
```



### 文件统计

```shell
# 统计当前文件夹下的文件行数
find . -type f | xargs cat | wc -l
# 统计当前文件夹下，去除vender文件夹的文件行数
find . -not -path '**/vender/**' -type f | xargs cat | wc -l
```

### 目录大小统计

```shell
# 统计当前目录下所有的目录大小
du -h
# 统计当前目录大小
du -d 0 -h
# 统计指定目录大小
du <dir_name> -d 0 -h
```



### 定时任务

```shell
# 编辑定时任务
crontab -e
# 分、时、日、月、周
# 例：0 2 * * * /root/cron/restart.sh >> /root/cron/restart.log
# 例：每小时第5分钟和第35分钟 上午8-11点 每隔两天 执行脚本
# 5,35 8-11 */2 * * /home/daily_task/write_task.sh >> write_task.log
# 例：每周一和周二晚上21:30分执行脚本
# 30 21 * * * 1,2 /home/daily_task/write_task.sh >> write_task.log
# 例：每月1,5,15号的晚上21:30执行脚本
# 30 21 * 1,5,15 * /home/daily_task/write_task.sh >> write_task.log
# * 取值范围内的所有数字
# / 每过多少个数字
# - 从X到Z
# ，散列数字

# 列出所有定时任务
crontab -ls
# 删除定时任务
crontab -r
```



### Git升级

```shell
# git版本 https://mirrors.edge.kernel.org/pub/software/scm/git/

yum remove git -y
yum -y install curl-devel expat-devel gettext-devel openssl-devel zlib-devel asciidoc
yum -y install gcc perl-ExtUtils-MakeMaker
cd /usr/local/src/
wget https://www.kernel.org/pub/software/scm/git/git-2.29.3.tar.xz
tar -vxf git-2.29.3.tar.xz
cd git-2.29.3
make prefix=/usr/local/git all
make prefix=/usr/local/git install
echo "export PATH=$PATH:/usr/local/git/bin" >> /etc/profile
source /etc/profile
git --version

# 注意：如果还是原来的版本，也不一定是安装失败了，首先检查一下Git的安装路径，可能有多个路径，是不是原来的Git版本没有删除，或者设置环境变量时添加了错误的路径。

# 查看Git的可执行文件位置，可能有多个路径
whereis git

# 再删除一遍
yum remove git -y
```

### Git操作

```shell
# 拉取代码
git clone http://10.20.5.5:10080/ehualu/Lakestor.git
git clone http://192.168.1.28/storage/storConsole.git --depth 0
# 查看所有分支
git branch -a
# 切换分支到feature-shell-v1
git checkout feature-shell-v1
# 查看当前分支，包含本地所有分支
git branch
# 更新远程分支索引
git fetch

# 查看修改内容
git status
# 查看具体修改内容
git diff <filename>
# 增加所有修改内容，若非全部，一个一个添加文件
git add .
# 增加备注
git commit -m "1.脚本增加权限;2.更新ceph.spec.in"
# 更新本地分支到最新版本
git pull origin feature-shell-v1
# 本地更新推送至远程分支
git push
# ntpdate cn.pool.ntp.org

### 还原操作

# 只修改了文件，未进行git操作
# 还原指定文件
git checkout aaa.html
# 还原本地所有修改
git checkout .

# 修改了文件，提交到暂存区(git add 已操作)
git log --online
# 回退到当前版本
git reset HEAD
git checkout aaa.html

# 还原最近一次提交的修改
git revert HEAD
# 还原指定版本的修改
git revert commit-id

# 修改文件，提交到仓库区(git commit -m "" 已操作)
git log --online
git reset HEAD^
git checkout -- aaa.html

# revert和reset的区别：reset是将head往后退而revert执行后head继续前行


# 合并feature_osd_update分支到master
git checkout master
git merge feature_osd_update
git push # ?
```

### gcc

```Shell
# 查看当前窗口的版本
gcc --version
# 当前窗口切换到devtoolset-8
scl enable devtoolset-8 bash
```

