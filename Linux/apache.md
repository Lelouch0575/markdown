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

## 虚拟主机

### 基于ip地址

#### 网卡绑定多个IP

使用nmtui为网卡配置3个ip

- 192.168.109.129

- 192.168.109.127

- 192.168.109.126

***

1. 分别在/home/wwwroot中创建用于保存不同网站数据的3个目录，并向其中分别写入网站的首页文件。每个首页文件中应有明确区分不同网站内容的信息，方便我们稍后能更直观地检查效果。
```
[root@linuxprobe ~]# mkdir -p /home/wwwroot/10
[root@linuxprobe ~]# mkdir -p /home/wwwroot/20
[root@linuxprobe ~]# mkdir -p /home/wwwroot/30
[root@linuxprobe ~]# echo "IP:192.168.10.10" > /home/wwwroot/10/index.html
[root@linuxprobe ~]# echo "IP:192.168.10.20" > /home/wwwroot/20/index.html
[root@linuxprobe ~]# echo "IP:192.168.10.30" > /home/wwwroot/30/index.html
```

2. 在httpd服务的配置文件中大约113行处开始，分别追加写入三个基于IP地址的虚拟主机网站参数，然后保存并退出。记得需要重启httpd服务

```
[root@linuxprobe ~]# vim /etc/httpd/conf/httpd.conf
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
[root@linuxprobe ~]# systemctl restart httpd
```

### 基于主机域名

1. 手工定义IP地址与域名之间对应关系的配置文件，保存并退出后会立即生效。可以通过分别ping这些域名来验证域名是否已经成功解析为IP地址。
```
[root@linuxprobe ~]# vim /etc/hosts
127.0.0.1 localhost localhost.localdomain localhost4 localhost4.localdomain4
::1 localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.10.10 www.linuxprobe.com bbs.linuxprobe.com tech.linuxprobe.com
[root@linuxprobe ~]# ping -c 4 www.linuxprobe.com
```
2. 分别在/home/wwwroot中创建用于保存不同网站数据的三个目录，并向其中分别写入网站的首页文件。每个首页文件中应有明确区分不同网站内容的信息，方便我们稍后能更直观地检查效果。

```
[root@linuxprobe ~]# mkdir -p /home/wwwroot/www
[root@linuxprobe ~]# mkdir -p /home/wwwroot/bbs
[root@linuxprobe ~]# mkdir -p /home/wwwroot/tech
[root@linuxprobe ~]# echo "WWW.linuxprobe.com" > /home/wwwroot/www/index.html
[root@linuxprobe ~]# echo "BBS.linuxprobe.com" > /home/wwwroot/bbs/index.html
[root@linuxprobe ~]# echo "TECH.linuxprobe.com" > /home/wwwroot/tech/index.html
```

3. 在httpd服务的配置文件中大约113行处开始，分别追加写入三个基于主机名的虚拟主机网站参数，然后保存并退出。记得需要重启httpd服务，这些配置才生效。

```
[root@linuxprobe ~]# vim /etc/httpd/conf/httpd.conf
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

### 基于端口号

1. 分别在`/home/wwwroot`中创建用于保存不同网站数据的两个目录，并向其中分别写入网站的首页文件。每个首页文件中应有明确区分不同网站内容的信息，方便我们稍后能更直观地检查效果。

```
[root@linuxprobe ~]# mkdir -p /home/wwwroot/6111
[root@linuxprobe ~]# mkdir -p /home/wwwroot/6222
[root@linuxprobe ~]# echo "port:6111" > /home/wwwroot/6111/index.html
[root@linuxprobe ~]# echo "port:6222" > /home/wwwroot/6222/index.html
```

2. 在httpd服务配置文件的第43行和第44行分别添加用于监听6111和6222端口的参数。

```
[root@linuxprobe ~]# vim /etc/httpd/conf/httpd.conf 
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
[root@linuxprobe ~]# vim /etc/httpd/conf/httpd.conf
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



## 访问控制

`Order Allow, Deny`表示先将源主机与允许规则进行匹配，若匹配成功则允许访问请求，反之则拒绝访问请求。 

1. 基于ip的访问控制访问

例如，我们只允许IP地址为192.168.10.20的主机访问网站资源，那么就可以在httpd服务配置文件的第129行后面添加下述规则

```
[root@linuxprobe ~]# vim /etc/httpd/conf/httpd.conf
………………省略部分输出信息………………
<Directory "/var/www/html/server">
Order allow,deny 
Allow from 192.168.10.20
</Directory>
………………省略部分输出信息………………
[root@linuxprobe ~]# systemctl restart httpd
[root@linuxprobe ~]# firefox
```

