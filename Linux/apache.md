# Apache

## 安装
安装Apache服务程序

`yum install httpd`

启用httpd服务程序并将其加入到开机启动项中

```
systemctl start httpd
systemctl enable httpd
```

在浏览器（这里以Firefox浏览器为例）的地址栏中输入`http://127.0.0.1`并按回车键，就可以看到用于提供Web服务的httpd服务程序的默认页面。

## 配置服务文件参数

|表 |Linux系统中的配置文件|
|--|--|
|服务目录 | /etc/httpd |
|主配置文件	|/etc/httpd/conf/httpd.conf|
|网站数据目录|	/var/www/html |
|访问日志	 | /var/log/httpd/access_log |
|错误日志	|/var/log/httpd/error_log |

|表|配置httpd服务程序时最常用的参数以及用途描述|
|--|--|
|ServerRoot|	服务目录|
|ServerAdmin|	管理员邮箱|
|User|	运行服务的用户|
|Group|	运行服务的用户组|
|ServerName|	网站服务器的域名|
|DocumentRoot|	网站数据目录|
|Listen|	监听的IP地址与端口号|
|DirectoryIndex|	默认的索引页页面|
|ErrorLog|	错误日志文件|
|CustomLog|	访问日志文件|
|Timeout|	网页超时时间，默认为300秒|

DocumentRoot参数用于定义网站数据的保存路径，其参数的默认值是把网站数据存放到/var/www/html目录中；而当前网站普遍的首页面名称是index.html。向/var/www/html目录中写入一个文件，替换掉httpd服务程序的默认首页面：

`# echo "Welcome To LinuxProbe.Com" > /var/www/html/index.html`

把保存网站数据的目录修改为/home/wwwroot目录：

```
# mkdir /home/wwwroot
# echo "The New Web Directory" > /home/wwwroot/index.html
# vim /etc/httpd/conf/httpd.conf
......
119 DocumentRoot "/home/wwwroot"
120 
121 #
122 # Relax access to content within /var/www.
123 #
124 <Directory "/home/wwwroot">
......
```
重新启动httpd服务程序并验证效果，发现页面中显示“Forbidden,You don't have permission to access /index.html on this server.”。这是SELinux的锅。

## SELinux

SELinux服务有三种配置模式

>enforcing：强制启用安全策略模式，将拦截服务的不合法请求。
>permissive：遇到服务越权访问时，只发出警告而不强制拦截。
>disabled：对于越权的行为不警告也不拦截。

```
# vim /etc/selinux/config
......
SELINUX=enforcing
......
SELINUXTYPE=targeted
```

使用`getenforce`命令获得当前SELinux服务的运行模式，用`setenforce [0|1]`命令临时修改SELinux当前的运行模式

查看原始网站数据的保存目录与当前网站数据的保存目录是否拥有不同的SELinux安全上下文值：

```
# setenforce 1
# ls -Zd /var/www/html
drwxr-xr-x. root root system_u:object_r:httpd_sys_content_t:s0 /var/www/html
# ls -Zd /home/wwwroot
drwxrwxrwx. root root unconfined_u:object_r:home_root_t:s0 /home/wwwroot
```

SELinux安全上下文是由**用户段**、**角色段**以及**类型段**等多个信息项共同组成的。用户段**system_u**代表系统进程的身份，角色段**object_r**代表文件目录的角色，类型段**httpd_sys_content_t**代表网站服务的系统文件。

>**semanage命令**
>
>semanage命令用于管理SELinux的策略，格式为“semanage [选项] [文件]”。
>
>-l参数用于查询；
>
>-a参数用于添加；
>
>-m参数用于修改；
>
>-d参数用于删除

还需要使用restorecon命令将设置好的SELinux安全上下文立即生效：

```
# restorecon -Rv /home/wwwroot/
restorecon reset /home/wwwroot context unconfined_u:object_r:home_root_t:s0->unconfined_u:object_r:httpd_sys_content_t:s0
restorecon reset /home/wwwroot/index.html context unconfined_u:object_r:home_root_t:s0->unconfined_u:object_r:httpd_sys_content_t:s0
```

## 个人用户主页功能

**第1步**开启个人用户主页

```
# vim /etc/httpd/conf.d/userdir.conf 
...... 
 17 # UserDir disabled
......
 24   UserDir public_html
......
```

**第2步**：在用户家目录中建立用于保存网站数据的目录及首页面文件

```
# su - linuxprobe
Last login: Fri May 22 13:17:37 CST 2017 on :0
$ mkdir public_html
$ echo "This is linuxprobe's website" > public_html/index.html
$ chmod -Rf 755 /home/linuxprobe
```

**第3步**：重新启动httpd服务程序，浏览器的地址栏中输入网址，其格式为“网址/~用户名”

`127.0.0.1/~linuxprobe`

**第4步**：配置SELinux安全策略

```
# getsebool -a | grep http
......
# setsebool -P httpd_enable_homedirs=on
```

### 口令功能

**第1步**：先使用htpasswd命令生成密码数据库。-c参数表示第一次生成

```
# htpasswd -c /etc/httpd/passwd linuxprobe
New password:此处输入用于网页验证的密码
Re-type new password:再输入一遍进行确认
Adding password for user linuxprobe
```

**第2步**：编辑个人用户主页功能的配置文件。

```
# vim /etc/httpd/conf.d/userdir.conf
27 #
28 # Control access to UserDir directories. The following is an example
29 # for a site where these directories are restricted to read-only.
30 #
31 <Directory "/home/*/public_html">
32 AllowOverride all
#刚刚生成出来的密码验证文件保存路径
33 authuserfile "/etc/httpd/passwd"
#当用户尝试访问个人用户网站时的提示信息
34 authname "My privately website"
35 authtype basic
#用户进行账户密码登录时需要验证的用户名称
36 require user linuxprobe
37 </Directory>
[root@linuxprobe ~]# systemctl restart httpd
```

## 虚拟主机

### 基于ip地址

#### 网卡绑定多个IP

使用nmtui为网卡配置3个ip

- 192.168.109.129

- 192.168.109.127

- 192.168.109.126

***

1. 分别在/home/wwwroot中创建用于保存不同网站数据的3个目录，并向其中分别写入网站的首页文件。

```
# mkdir -p /home/wwwroot/10
# mkdir -p /home/wwwroot/20
# mkdir -p /home/wwwroot/30
# echo "IP:192.168.10.10" > /home/wwwroot/10/index.html
# echo "IP:192.168.10.20" > /home/wwwroot/20/index.html
# echo "IP:192.168.10.30" > /home/wwwroot/30/index.html
```

2. 在httpd服务的配置文件中大约113行处开始，分别追加写入三个基于IP地址的虚拟主机网站参数，重启httpd服务。

```
# vim /etc/httpd/conf/httpd.conf

………………省略部分输出信息………………
<VirtualHost 192.168.10.10>
DocumentRoot /home/wwwroot/10
ServerName www.linuxprobe.com
<Directory /home/wwwroot/10 >
AllowOverride None
Require all granted
</Directory>
</VirtualHost>
<VirtualHost 192.168.10.20>
DocumentRoot /home/wwwroot/20
ServerName bbs.linuxprobe.com
<Directory /home/wwwroot/20 >
AllowOverride None
Require all granted
</Directory>
</VirtualHost>
<VirtualHost 192.168.10.30>
DocumentRoot /home/wwwroot/30
ServerName tech.linuxprobe.com
<Directory /home/wwwroot/30 >
AllowOverride None
Require all granted
</Directory>
</VirtualHost>
………………省略部分输出信息………………

# systemctl restart httpd
```

3.  设置SELinux安全上下文

```
# semanage fcontext -a -t httpd_sys_content_t /home/wwwroot
# semanage fcontext -a -t httpd_sys_content_t /home/wwwroot/10
# semanage fcontext -a -t httpd_sys_content_t /home/wwwroot/10/*
# semanage fcontext -a -t httpd_sys_content_t /home/wwwroot/20
# semanage fcontext -a -t httpd_sys_content_t /home/wwwroot/20/*
# semanage fcontext -a -t httpd_sys_content_t /home/wwwroot/30
# semanage fcontext -a -t httpd_sys_content_t /home/wwwroot/30/*
# restorecon -Rv /home/wwwroot
```

### 基于主机域名

1. 手工定义IP地址与域名之间对应关系的配置文件，保存并退出后会立即生效。可以通过分别ping这些域名来验证域名是否已经成功解析为IP地址。
```
# vim /etc/hosts
127.0.0.1 localhost localhost.localdomain localhost4 localhost4.localdomain4
::1 localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.10.10 www.linuxprobe.com bbs.linuxprobe.com tech.linuxprobe.com
# ping -c 4 www.linuxprobe.com
```
2. 分别在/home/wwwroot中创建用于保存不同网站数据的三个目录，并向其中分别写入网站的首页文件。

```
# mkdir -p /home/wwwroot/www
# mkdir -p /home/wwwroot/bbs
# mkdir -p /home/wwwroot/tech
# echo "WWW.linuxprobe.com" > /home/wwwroot/www/index.html
# echo "BBS.linuxprobe.com" > /home/wwwroot/bbs/index.html
# echo "TECH.linuxprobe.com" > /home/wwwroot/tech/index.html
```

3. 在httpd服务的配置文件中大约113行处开始，分别追加写入三个基于主机名的虚拟主机网站参数，重启httpd服务。

```
# vim /etc/httpd/conf/httpd.conf

………………省略部分输出信息………………
113 <VirtualHost 192.168.10.10>
114 DocumentRoot "/home/wwwroot/www"
115 ServerName "www.linuxprobe.com"
116 <Directory "/home/wwwroot/www">
117 AllowOverride None
118 Require all granted
119 </directory> 
120 </VirtualHost>
121 <VirtualHost 192.168.10.10>
122 DocumentRoot "/home/wwwroot/bbs"
123 ServerName "bbs.linuxprobe.com"
124 <Directory "/home/wwwroot/bbs">
125 AllowOverride None
126 Require all granted
127 </Directory>
128 </VirtualHost>
129 <VirtualHost 192.168.10.10>
130 DocumentRoot "/home/wwwroot/tech"
131 ServerName "tech.linuxprobe.com"
132 <Directory "/home/wwwroot/tech">
133 AllowOverride None
134 Require all granted
135 </directory>
136 </VirtualHost>
………………省略部分输出信息………………
```

4. 设置SELinux安全上下文

```
# semanage fcontext -a -t httpd_sys_content_t /home/wwwroot
# semanage fcontext -a -t httpd_sys_content_t /home/wwwroot/www
# semanage fcontext -a -t httpd_sys_content_t /home/wwwroot/www/*
# semanage fcontext -a -t httpd_sys_content_t /home/wwwroot/bbs
# semanage fcontext -a -t httpd_sys_content_t /home/wwwroot/bbs/*
# semanage fcontext -a -t httpd_sys_content_t /home/wwwroot/tech
# semanage fcontext -a -t httpd_sys_content_t /home/wwwroot/tech/*
# restorecon -Rv /home/wwwroot
```

### 基于端口号

1. 分别在`/home/wwwroot`中创建用于保存不同网站数据的两个目录，并向其中分别写入网站的首页文件。每个首页文件中应有明确区分不同网站内容的信息，方便我们稍后能更直观地检查效果。

```
# mkdir -p /home/wwwroot/6111
# mkdir -p /home/wwwroot/6222
# echo "port:6111" > /home/wwwroot/6111/index.html
# echo "port:6222" > /home/wwwroot/6222/index.html
```

2. 在httpd服务配置文件的第43行和第44行分别添加用于监听6111和6222端口的参数。

```
# vim /etc/httpd/conf/httpd.conf 

………………省略部分输出信息……………… 
 33 #
 34 # Listen: Allows you to bind Apache to specific IP addresses and/or
 35 # ports, instead of the default. See also the <VirtualHost>
 36 # directive.
 37 #
 38 # Change this to Listen on specific IP addresses as shown below to 
 39 # prevent Apache from glomming onto all bound IP addresses.
 40 #
 41 #Listen 12.34.56.78:80
 42 Listen 80
 43 Listen 6111
 44 Listen 6222
………………省略部分输出信息……………… 
```

3. 在httpd服务的配置文件中大约113行处开始，分别追加写入两个基于端口号的虚拟主机网站参数，然后保存并退出。记得需要重启httpd服务，这些配置才生效。

```
# vim /etc/httpd/conf/httpd.conf

………………省略部分输出信息……………… 
113 <VirtualHost 192.168.10.10:6111>
114 DocumentRoot "/home/wwwroot/6111"
115 ServerName www.linuxprobe.com
116 <Directory "/home/wwwroot/6111">
117 AllowOverride None
118 Require all granted
119 </Directory> 
120 </VirtualHost>
121 <VirtualHost 192.168.10.10:6222>
122 DocumentRoot "/home/wwwroot/6222"
123 ServerName bbs.linuxprobe.com
124 <Directory "/home/wwwroot/6222">
125 AllowOverride None
126 Require all granted
127 </Directory>
128 </VirtualHost>
………………省略部分输出信息………………
```

4. SELinux安全上下文

```
# semanage fcontext -a -t httpd_sys_content_t /home/wwwroot
# semanage fcontext -a -t httpd_sys_content_t /home/wwwroot/6111
# semanage fcontext -a -t httpd_sys_content_t /home/wwwroot/6111/*
# semanage fcontext -a -t httpd_sys_content_t /home/wwwroot/6222
# semanage fcontext -a -t httpd_sys_content_t /home/wwwroot/6222/*
# restorecon -Rv /home/wwwroot/
```

5. SELinux允许的与HTTP协议相关的端口号中默认没有包含6111和6222，因此需要将这两个端口号手动添加进去。

```
# semanage port -l | grep http

# semanage port -a -t http_port_t -p tcp 6111
# semanage port -a -t http_port_t -p tcp 6222
# semanage port -l | grep http
# systemctl restart httpd
```

## 访问控制

`Order Allow, Deny`表示先将源主机与允许规则进行匹配，若匹配成功则允许访问请求，反之则拒绝访问请求。 

### 基于源主机上的浏览器特征

在服务器上的网站数据目录中新建一个子目录，并在这个子目录中创建一个包含Successful单词的首页文件

```
# mkdir /var/www/html/server
# echo "Successful" > /var/www/html/server/index.html
```

打开httpd服务的配置文件，在第129行后面添加下述规则来限制源主机的访问。这段规则的含义是允许使用Firefox浏览器的主机访问服务器上的首页文件，除此之外的所有请求都将被拒绝。

```
# vim /etc/httpd/conf/httpd.conf

………………省略部分输出信息………………
129 <Directory "/var/www/html/server">
130 SetEnvIf User-Agent "Firefox" ff=1
131 Order allow,deny
132 Allow from env=ff
133 </Directory>
………………省略部分输出信息………………

# systemctl restart httpd
```

### 基于ip的访问控制访问

例如，我们只允许IP地址为192.168.10.20的主机访问网站资源，那么就可以在httpd服务配置文件的第129行后面添加下述规则

```
# vim /etc/httpd/conf/httpd.conf

………………省略部分输出信息………………
<Directory "/var/www/html/server">
Order allow,deny 
Allow from 192.168.10.20
</Directory>
………………省略部分输出信息………………

# systemctl restart httpd
# firefox
```

