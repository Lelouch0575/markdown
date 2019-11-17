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

配置文件`/etc/samba/smb.conf`，示例：

```
[database]  #共享名为database
	comment = samba share
	path = /smbfile  #共享路径
	browseable = yes #指定共享信息是否在“网上邻居”中可见
	writeable = yes  #允许写入
	public = no #关闭“所有人可见”
```

### 1. 创建用于访问共享资源的账户信息

pdbedit命令用于管理SMB服务程序的账户信息数据库。在第一次把账户信息写入到数据库时需要使用-a参数，以后在执行修改密码、删除账户等操作时就不再需要该参数了

```
pdbedit -a username：新建Samba账户。
pdbedit -r username：修改Samba账户。
pdbedit -x username：删除Samba账户。
pdbedit -L：列出Samba用户列表，读取passdb.tdb数据库文件。
pdbedit -Lv：列出Samba用户列表详细信息。
pdbedit -c “[D]” -u username：暂停该Samba用户账号。
pdbedit -c “[]” -u username：恢复该Samba用户账号
```

```
smbpasswd 属于samba套件，能够实现添加或删除samba用户和为用户修改密码
参数：
 -a：向smbpasswd文件中添加用户；
 -c：指定samba的配置文件；
 -x：从smbpasswd文件中删除用户；
 -d：在smbpasswd文件中禁用指定的用户；
 -e：在smbpasswd文件中激活指定的用户；
 -n：将指定的用户的密码置空。
```

```
# id linuxprobe
uid=1000(linuxprobe) gid=1000(linuxprobe) groups=1000(linuxprobe)
# pdbedit -a -u linuxprobe
new password:此处输入该账户在Samba服务数据库中的密码
retype new password:再次输入密码进行确认
Unix username: linuxprobe
......
```

### 2. 创建用于共享资源的文件目录

```
# mkdir /home/database
# chown -Rf linuxprobe:linuxprobe /home/database
# semanage fcontext -a -t samba_share_t /home/database
# restorecon -Rv /home/database
```

### 3. 设置SELinux服务与策略

```
# getsebool -a | grep samba
......
# setsebool -P samba_enable_home_dirs on
# setsebool -P samba_export_all_rw on
```

### 4. 写入共享信息

在原始的配置文件中，[homes]参数为来访用户的家目录共享信息，[printers]参数为共享的打印机设备。

```
# vim /etc/samba/smb.conf 
[global]
 workgroup = MYGROUP
 server string = Samba Server Version %v
 log file = /var/log/samba/log.%m
 max log size = 50
 security = user
 passdb backend = tdbsam
 load printers = yes
 cups options = raw
[database]
 comment = Do not arbitrarily modify the database file
 path = /home/database
 public = no
 writable = yes
```

### 5. 清空iptables防火墙

```
# systemctl restart smb
# systemctl enable smb
 ln -s '/usr/lib/systemd/system/smb.service' '/etc/systemd/system/multi-user.target.wants/smb.service'
# iptables -F
# service iptables save
iptables: Saving firewall rules to /etc/sysconfig/iptables:[ OK ]
```

## Linux挂载

### 永久挂载

`yum install cifs-utils`

在Linux客户端，按照Samba服务的用户名、密码、共享域的顺序将相关信息写入到一个认证文件中。为了保证不被其他人随意看到，最后把这个认证文件的权限修改为仅root管理员才能够读写：

```
# vim auth.smb
username=linuxprobe
password=redhat
domain=MYGROUP
# chmod -Rf 600 auth.smb
```

在Linux客户端上创建一个用于挂载Samba服务共享资源的目录，并把挂载信息写入到`/etc/fstab`文件中，以确保共享挂载信息在服务器重启后依然生效：

```
# mkdir /database
# vim /etc/fstab
......
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

>- 附：
>
>启动服务
>`systemctl start smb`
>
>关闭并禁用防火墙
>
>`systemctl stop fillwalld`
>
>`systemctl disable fillwalld`

# NFS

## 安装

`# yum install nfs-utils`

## 配置

### 1. 清空NFS服务器上面iptables防火墙

```
# iptables -F
# service iptables save
iptables: Saving firewall rules to /etc/sysconfig/iptables:[ OK ]
```

### 2. 建立用于NFS文件共享的目录，并设置足够的权限

```
# mkdir /nfsfile
# chmod -Rf 777 /nfsfile
# echo "welcome to linuxprobe.com" > /nfsfile/readme
```

### 3. 修改配置文件/etc/exports

格式：“共享目录的路径 允许访问的NFS客户端（共享权限参数）”

|参数|	作用|
|--|--|
|ro|	只读|
|rw|	读写|
|root_squash|	当NFS客户端以root管理员访问时，映射为NFS服务器的匿名用户|
|no_root_squash|	当NFS客户端以root管理员访问时，映射为NFS服务器的root管理员|
|all_squash|	无论NFS客户端使用什么账户访问，均映射为NFS服务器的匿名用户|
|sync	|同时将数据写入到内存与硬盘中，保证不丢失数据|
|async|	优先将数据保存到内存，然后再写入硬盘；这样效率更高，但可能会丢失数据|

 ***注意，NFS客户端地址与权限之间没有空格。***

```
# vim /etc/exports
/nfsfile 192.168.10.*(rw,sync,root_squash)
```

### 4. 启动和启用NFS服务程序

```
# systemctl restart rpcbind
# systemctl enable rpcbind
# systemctl start nfs-server
# systemctl enable nfs-server
```

使用showmount命令查询NFS服务器的远程共享信息：

|参数|	作用|
|--|--|
|-e|	显示NFS服务器的共享列表|
|-a|	显示本机挂载的文件资源的情况NFS资源的情况|
|-v|	显示版本号|

```
# showmount -e 192.168.10.10
Export list for 192.168.10.10:
/nfsfile 192.168.10.*
```

挂载目录：

```
# mkdir /nfs
# mount -t nfs 192.168.10.10:/nfsfile /nfs
```

写入到fstab文件，使挂载一直有效

```
# vim /etc/fstab 
......
/dev/mapper/rhel-root / xfs defaults 1 1
UUID=812b1f7c-8b5b-43da-8c06-b9999e0fe48b /boot xfs defaults 1 2
/dev/mapper/rhel-swap swap swap defaults 0 0
/dev/cdrom /media/cdrom iso9660 defaults 0 0 
192.168.10.10:/nfsfile /nfsfile nfs defaults 0 0
```
