# samba

## 安装

`yum install samba`

`//yum -y install samba samba-client samba-common`

## 配置文件`/etc/samba/smb.conf`

```
[global]		#全局参数。
	workgroup = MYGROUP	#工作组名称
	server string = Samba Server Version %v	#服务器介绍信息，参数%v为显示SMB版本号
	log file = /var/log/samba/log.%m	#定义日志文件的存放位置与名称，参数%m为来访的主机名
	max log size = 50	#定义日志文件的最大容量为50KB
	security = user	#安全验证的方式，总共有4种
	#share：来访主机无需验证口令；比较方便，但安全性很差
	#user：需验证来访主机提供的口令后才可以访问；提升了安全性
	#server：使用独立的远程主机验证来访主机提供的口令（集中管理账户）
	#domain：使用域控制器进行身份验证
	passdb backend = tdbsam	#定义用户后台的类型，共有3种
	#smbpasswd：使用smbpasswd命令为系统用户设置Samba服务程序的密码
	#tdbsam：创建数据库文件并使用pdbedit命令建立Samba服务程序的用户
	#ldapsam：基于LDAP服务进行账户验证
	load printers = yes	#设置在Samba服务启动时是否共享打印机设备
	cups options = raw	#打印机的选项
[homes]		#共享参数
	comment = Home Directories	#描述信息
	browseable = no	#指定共享信息是否在“网上邻居”中可见
	writable = yes	#定义是否可以执行写入操作，与“read only”相反
[printers]		#打印机共享参数
	comment = All Printers	
	path = /var/spool/samba	#共享文件的实际路径(重要)。
	browseable = no	
	guest ok = no	#是否所有人可见，等同于"public"参数。
	writable = no	
	printable = yes
```

## 配置共享资源

配置`/etc/samba/smb.conf`，在末尾追加内容：

```
[database]  #共享名为database
	comment = samba share
	path = /smbfile  #共享路径
	browseable = yes #指定共享信息是否在“网上邻居”中可见
	writeable = yes  #允许写入
	public = no #关闭“所有人可见”
```

### 创建用于访问共享资源的账户信息

pdbedit命令用于管理SMB服务程序的账户信息数据库，格式为“pdbedit [选项] 账户”。在第一次把账户信息写入到数据库时需要使用-a参数，以后在执行修改密码、删除账户等操作时就不再需要该参数了

|参数|	作用|
|--|--|
|-a 用户名|	建立Samba用户|
|-x 用户名|	删除Samba用户|
|-L|	列出用户列表|
|-Lv	|列出用户详细信息的列表|

```
[root@linuxprobe ~]# id linuxprobe
uid=1000(linuxprobe) gid=1000(linuxprobe) groups=1000(linuxprobe)
[root@linuxprobe ~]# pdbedit -a -u linuxprobe
new password:此处输入该账户在Samba服务数据库中的密码
retype new password:再次输入密码进行确认
Unix username: linuxprobe
......
```

### 创建用于共享资源的文件目录

```
mkdir /home/database
chown -Rf /home/database
```

### 清空iptables防火墙
```
[root@linuxprobe ~]# systemctl restart smb
[root@linuxprobe ~]# systemctl enable smb
 ln -s '/usr/lib/systemd/system/smb.service' '/etc/systemd/system/multi-user.target.wants/smb.service'
[root@linuxprobe ~]# iptables -F
[root@linuxprobe ~]# service iptables save
iptables: Saving firewall rules to /etc/sysconfig/iptables:[ OK ]
```

## Linux挂载

### 永久挂载

`yum install cifs-utils`

在Linux客户端，按照Samba服务的用户名、密码、共享域的顺序将相关信息写入到一个认证文件中。为了保证不被其他人随意看到，最后把这个认证文件的权限修改为仅root管理员才能够读写：

```
[root@linuxprobe ~]# vim auth.smb
username=linuxprobe
password=redhat
domain=MYGROUP
[root@linuxprobe ~]# chmod -Rf 600 auth.smb
```

现在，在Linux客户端上创建一个用于挂载Samba服务共享资源的目录，并把挂载信息写入到`/etc/fstab`文件中，以确保共享挂载信息在服务器重启后依然生效：

```
[root@linuxprobe ~]# mkdir /database
[root@linuxprobe ~]# vim /etc/fstab
#
# /etc/fstab
# Created by anaconda on Wed May 4 19:26:23 2017
#
# Accessible filesystems, by reference, are maintained under '/dev/disk'
# See man pages fstab(5), findfs(8), mount(8) and/or blkid(8) for more info
#
/dev/mapper/rhel-root / xfs defaults 1 1
UUID=812b1f7c-8b5b-43da-8c06-b9999e0fe48b /boot xfs defaults 1 2
/dev/mapper/rhel-swap swap swap defaults 0 0
/dev/cdrom /media/cdrom iso9660 defaults 0 0 
//192.168.10.10/database /database cifs credentials=/root/auth.smb 0 0
[root@linuxprobe ~]# mount -a
```

### 临时挂载

```
尝试链接是否成功
smbclient -L 192.168.1.1/dir -U username

创建本地文件夹,共享目录
mkdir /home/user/dir

挂在远程文件夹到本地共享目录
mount -t cifs -o username=xxx,password=xxx //192.168.1.1/dir /home/user/dir

```



- 附：

启动服务
`systemctl start smb`

关闭并禁用防火墙

`systemctl stop fillwalld`

`systemctl disable fillwalld`

