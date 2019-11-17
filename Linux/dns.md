# DNS

## 安装Bind

`yum install bind-chroot`

在bind服务程序中有下面这三个比较关键的文件：

> **主配置文件（/etc/named.conf）**：只有58行，而且在去除注释信息和空行之后，实际有效的参数仅有30行左右，这些参数用来定义bind服务程序的运行。
**区域配置文件（/etc/named.rfc1912.zones）**：用来保存域名和IP地址对应关系的所在位置。类似于图书的目录，对应着每个域和相应IP地址所在的具体位置，当需要查看或修改时，可根据这个位置找到相关文件。
**数据配置文件目录（/var/named）**：该目录用来保存域名和IP地址真实对应关系的数据配置文件。

bind服务程序的名称为named。

在/etc目录中找到该服务程序的主配置文件，然后把第11行和第17行的地址均修改为any，分别表示服务器上的所有IP地址均可提供DNS域名解析服务。

```
# vim /etc/named.conf
...
11 listen-on port 53 { any; };
...
17 allow-query { any; };
...
```
*可以执行named-checkconf命令和named-checkzone命令，分别检查主配置文件与数据配置文件中语法或参数的错误。*

### 检查DNS（BIND）配置文件
#### 1：检查DNS（绑定）配置
`# named-checkconf /etc/named.conf`
如果绑定在chroot环境中运行使用下面的命令也随着上面的命令
`named-checkconf -t /var/named/chroot /etc/named.conf`
#### 2：检查绑定区域文件
`# named-checkzone demohowtoing.com /var/named/demohowtoing.com.db`
#### 3：检查配置文件中的绑定旧版本
`# service named configtest`

## 正向解析

正向解析是指根据域名（主机名）查找到对应的IP地址。

**第1步**：编辑区域配置文件。可以将下面的参数添加到区域配置文件的最下面，当然，也可以将该文件中的原有信息全部清空，而只保留自己的域名解析信息：

```
# vim /etc/named.rfc1912.zones
zone "linuxprobe.com" IN {
type master;
file "linuxprobe.com.zone";
allow-update {none;};
};
```

**第2步**：编辑数据配置文件。从/var/named目录中复制一份正向解析的模板文件（named.localhost）。在复制时记得加上-a参数，这可以保留原始文件的所有者、所属组、权限属性等信息，以便让bind服务程序顺利读取文件内容：

```
# cd /var/named/
# ls -al named.localhost
-rw-r-----. 1 root named 152 Jun 21 2007 named.localhost
# cp -a named.localhost linuxprobe.com.zone
```

编辑数据配置文件

```
# vim linuxprobe.com.zone
# systemctl restart named
```

```
$TTL 1D #生存周期为1天
@       IN SOA  linuxprobe.com. root.linuxprobe.com.     (
		#授权信息开始:	#DNS区域的地址	#域名管理员的邮箱(不要用@符号)
                                        0       ; serial    #更新序列号
                                        1D      ; refresh   #更新时间
                                        1H      ; retry     #重试延时
                                        1W      ; expire    #失效时间
                                        3H )    ; minimum   #无效解析记录的缓存时间
        NS      ns.linuxprobe.com.				#域名服务器记录
ns      IN A    192.168.109.129					#地址记录(ns.linuxprobe.com.)
        IN MX 10        mail.linuxprobe.com.	#邮箱交换记录
mail    IN A    192.168.109.129					#地址记录(mail.linuxprobe.com.)
www     IN A    192.168.109.129					#地址记录(www.linuxprobe.com.)
bbs     IN A    192.168.109.127					#地址记录(bbs.linuxprobe.com.)
```

**第3步**：检验解析结果。

把Linux系统网卡中的DNS地址参数修改成本机IP地址。

nslookup命令用于检测能否从DNS服务器中查询到域名与IP地址的解析记录：

```
# systemctl restart network
# nslookup
> www.linuxprobe.com
Server: 127.0.0.1
Address: 127.0.0.1#53
Name: www.linuxprobe.com
Address: 192.168.10.10
> bbs.linuxprobe.com
Server: 127.0.0.1
Address: 127.0.0.1#53
Name: bbs.linuxprobe.com
Address: 192.168.10.20
```

## 反向解析

反向解析的作用是将用户提交的IP地址解析为对应的域名信息。

**第1步**：编辑区域配置文件

在定义zone（区域）时应该要把IP地址反写，比如原来是192.168.10.0，反写后应该就是10.168.192，而且只需写出IP地址的网络位即可。把下列参数添加至正向解析参数的后面。

```
# vim /etc/named.rfc1912.zones
zone "linuxprobe.com" IN {
type master;
file "linuxprobe.com.zone";
allow-update {none;};
};
zone "10.168.192.in-addr.arpa" IN {
type master;
file "192.168.10.arpa";
};
```

**第2步**：编辑数据配置文件。首先从/var/named目录中复制一份反向解析的模板文件（named.loopback），然后把下面的参数填写到文件中。其中，IP地址仅需要写主机位，

```
10 IN	PTR		www.linuxprobe.com.
20 IN	PTR		bbs.linuxprobe.com.
#在192.168.10.in-addr.arpa反向区域文件中，则对应为192.168.10.20的ip地址
```

```
# cp -a named.loopback 192.168.10.arpa
# vim 192.168.10.arpa
# systemctl restart named
```

```
$TTL 1D
@       IN SOA  linuxprobe.com. root.linuxprobe.com. (
                                        0       ; serial
                                        1D      ; refresh
                                        1H      ; retry
                                        1W      ; expire
                                        3H )    ; minimum
        NS      ns.linuxprobe.com.
ns      A       192.168.109.129
10      PTR     ns.linuxprobe.com.		#PTR为指针记录，仅用于反向解析中。
10      PTR     mail.linuxprobe.com.
10      PTR     www.linuxprobe.com.
20      PTR     bbs.linuxprobe.com.
```

**第3步**：检验解析结果。

```
[root@linuxprobe ~]# nslookup
> 192.168.10.10
Server: 127.0.0.1
Address: 127.0.0.1#53
10.10.168.192.in-addr.arpa name = ns.linuxprobe.com.
10.10.168.192.in-addr.arpa name = www.linuxprobe.com.
10.10.168.192.in-addr.arpa name = mail.linuxprobe.com.
> 192.168.10.20
Server: 127.0.0.1
Address: 127.0.0.1#53
20.10.168.192.in-addr.arpa name = bbs.linuxprobe.com.
```
