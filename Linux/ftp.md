# FTP

## 概念

###  两种工作模式 

**主动模式**：FTP服务器主动向客户端发起连接请求。（port）

**被动模式**：FTP服务器等待客户端发起连接请求（FTP的默认工作模式）。（pasv）

### 两个TCP连接

控制连接

数据连接

### 传输模式

ASCII传输模式

二进制数据传输模式

## 配置

###  三种认证模式 

**匿名开放模式**：是一种最不安全的认证模式，任何人都可以无需密码验证而直接登录到FTP服务器。

**本地用户模式**：是通过Linux系统本地的账户密码信息进行认证的模式，相较于匿名开放模式更安全，而且配置起来也很简单。但是如果被黑客破解了账户的信息，就可以畅通无阻地登录FTP服务器，从而完全控制整台服务器。

**虚拟用户模式**：是这三种模式中最安全的一种认证模式，它需要为FTP服务单独建立用户数据库文件，虚拟出用来进行口令验证的账户信息，而这些账户信息在服务器系统中实际上是不存在的，仅供FTP服务程序进行认证使用。这样，即使黑客破解了账户信息也无法登录服务器，从而有效降低了破坏范围和影响。

安装：

 vsftpd（very secure ftp daemon，非常安全的FTP守护进程）是一款运行在Linux操作系统上的FTP服务程序 。

`yum install vsftpd`

清空iptables防火墙的默认策略，并把当前已经被清理的防火墙策略状态保存下来：

```
# iptables -F
# service iptables save
iptables: Saving firewall rules to /etc/sysconfig/iptables:[ OK ]
```

|表 |vsftpd服务程序常用的参数以及作用|
|--|--|
|参数|	作用|
|listen=[YES\|NO]	|是否以独立运行的方式监听服务|
|listen_address=IP地址|	设置要监听的IP地址|
|listen_port=21|	设置FTP服务的监听端口|
|download_enable＝[YES\|NO]|	是否允许下载文件|
|userlist_enable=[YES\|NO]   userlist_deny=[YES\|NO]|	设置用户列表为“允许”还是“禁止”操作|
|max_clients=0|	最大客户端连接数，0为不限制|
|max_per_ip=0|	同一IP地址的最大连接数，0为不限制|
|anonymous_enable=[YES\|NO]|	是否允许匿名用户访问||
|anon_upload_enable=[YES\|NO]|	是否允许匿名用户上传文件||
|anon_umask=022|	匿名用户上传文件的umask值|
|anon_root=/var/ftp|	匿名用户的FTP根目录|
|anon_mkdir_write_enable=[YES\|NO]|	是否允许匿名用户创建目录|
|anon_other_write_enable=[YES\|NO]|	是否开放匿名用户的其他写入权限（包括重命名、删除等操作权限）|
|anon_max_rate=0|	匿名用户的最大传输速率（字节/秒），0为不限制|
|local_enable=[YES\|NO]|	是否允许本地用户登录FTP|
|local_umask=022|	本地用户上传文件的umask值|
|local_root=/var/ftp|	本地用户的FTP根目录|
|chroot_local_user=[YES\|NO]|	是否将用户权限禁锢在FTP目录，以确保安全|
|local_max_rate=0|	本地用户最大传输速率（字节/秒），0为不限制|

### 匿名访问模式

 ftp是Linux系统中以命令行界面的方式来管理FTP传输服务的客户端工具。 

`yum install ftp`

|表 |可以向匿名用户开放的权限参数以及作用|
|--|--|
|参数	|作用 |
|anonymous_enable=YES|	允许匿名访问模式|
|anon_umask=022	|匿名用户上传文件的umask值|
|anon_upload_enable=YES	|允许匿名用户上传文件|
|anon_mkdir_write_enable=YES|	允许匿名用户创建目录|
|anon_other_write_enable=YES|	允许匿名用户修改目录名称或删除目录|

```
# vim /etc/vsftpd/vsftpd.conf
1 anonymous_enable=YES
2 anon_umask=022
3 anon_upload_enable=YES
4 anon_mkdir_write_enable=YES
5 anon_other_write_enable=YES
......
# systemctl restart vsftpd
# systemctl enable vsftpd
```

登录：`ftp 192.168.10.10`，用户名：`anonymous`，密码：无。默认访问的是`/var/ftp`目录，只有root管理员才有写入权限。

将`/var/ftp/pub`的所有者身份改成系统账户`ftp`（该账户在系统中已经存在）

```
# ls -ld /var/ftp/pub
drwxr-xr-x. 3 root root 16 Jul 13 14:38 /var/ftp/pub
# chown -Rf ftp /var/ftp/pub
# ls -ld /var/ftp/pub
drwxr-xr-x. 3 ftp root 16 Jul 13 14:38 /var/ftp/pub
```

```
# getsebool -a | grep ftp

# setsebool -P ftpd_full_access=on
```

### 本地用户模式

|参数|作用|
|--|--|
|anonymous_enable=NO|	禁止匿名访问模式|
|local_enable=YES|	允许本地用户模式|
|write_enable=YES|	设置可写权限|
|local_umask=022|	本地用户模式创建文件的umask值|
|userlist_deny=YES|	启用“禁止用户名单”，名单文件为ftpusers和user_list|
|userlist_enable=YES	|开启用户作用名单文件功能|

```
# vim /etc/vsftpd/vsftpd.conf
1 anonymous_enable=NO
2 local_enable=YES
3 write_enable=YES
4 local_umask=022
......
```

```
# setsebool -P ftpd_full_access=on
```

选择`/etc/vsftpd/ftpusers`和`/etc/vsftpd/user_list`文件中没有的一个普通用户尝试登录FTP服务器

本地用户模式登录FTP服务器后，默认访问的是该用户的家目录。

### 虚拟用户模式

虚拟用户模式是这三种模式中最安全的一种认证模式。

**第1步**：创建用于进行FTP认证的用户数据库文件，其中奇数行为账户名，偶数行为密码。

```
# cd /etc/vsftpd/
# vim vuser.list
zhangsan
redhat
lisi
redhat
```

使用db_load命令用哈希（hash）算法将原始的明文信息文件转换成数据库文件，并且降低数据库文件的权限（避免其他人看到数据库文件的内容），然后再把原始的明文信息文件删除。

```
# db_load -T -t hash -f vuser.list vuser.db
# file vuser.db
vuser.db: Berkeley DB (Hash, version 9, native byte-order)
# chmod 600 vuser.db
# rm -f vuser.list
```

**第2步**：创建vsftpd服务程序用于存储文件的根目录以及虚拟用户映射的系统本地用户。

```
# useradd -d /var/ftproot -s /sbin/nologin virtual
# ls -ld /var/ftproot/
drwx------. 3 virtual virtual 74 Jul 14 17:50 /var/ftproot/
# chmod -Rf 755 /var/ftproot/
```

**第3步**：建立用于支持虚拟用户的PAM文件。

```
# vim /etc/pam.d/vsftpd.vu
auth       required     pam_userdb.so db=/etc/vsftpd/vuser
account    required     pam_userdb.so db=/etc/vsftpd/vuser
```

**第4步**：在vsftpd服务程序的主配置文件中通过pam_service_name参数将PAM认证文件的名称修改为vsftpd.vu

|参数|	作用|
|--|--|
|anonymous_enable=NO|	禁止匿名开放模式|
|local_enable=YES|	允许本地用户模式|
|guest_enable=YES|	开启虚拟用户模式|
|guest_username=virtual|	指定虚拟用户账户|
|pam_service_name=vsftpd.vu|	指定PAM文件|
|allow_writeable_chroot=YES|	允许对禁锢的FTP根目录执行写入操作，而且不拒绝用户的登录请求|

```
# vim /etc/vsftpd/vsftpd.conf
1 anonymous_enable=NO
2 local_enable=YES
3 guest_enable=YES
4 guest_username=virtual
5 allow_writeable_chroot=YES
......
14 pam_service_name=vsftpd.vu
......
```

**第5步**：为虚拟用户设置不同的权限

```
# mkdir /etc/vsftpd/vusers_dir/
# cd /etc/vsftpd/vusers_dir/
# touch lisi
# vim zhangsan
anon_upload_enable=YES
anon_mkdir_write_enable=YES
anon_other_write_enable=YES
```

修改vsftpd主配置文件，通过添加user_config_dir参数来定义这两个虚拟用户不同权限的配置文件所存放的路径

```
# vim /etc/vsftpd/vsftpd.conf
......
user_config_dir=/etc/vsftpd/vusers_dir
```

**第6步**：设置SELinux域允许策略

```
# getsebool -a | grep ftp
# setsebool -P ftpd_full_access=on
```

