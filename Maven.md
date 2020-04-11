# Maven

## 环境变量

**MAVEN_HOME** is used for maven 1 and **M2_HOME** is used to locate maven 2.

Anyway usage of this pattern is now deprecated with maven 3.5 as per the [documentation](https://maven.apache.org/docs/3.5.0/release-notes.html).

> Based on problems in using M2_HOME related to different Maven versions installed and to simplify things, the usage of M2_HOME has been removed and is not supported any more [MNG-5823](https://issues.apache.org/jira/browse/MNG-5823), [MNG-5836](https://issues.apache.org/jira/browse/MNG-5836), [MNG-5607](https://issues.apache.org/jira/browse/MNG-5607)

So what you need is **JAVA_HOME** to be set correctly. As per the new installation guide, just add the maven bin directory path to the **PATH** variable. It should do the trick.

ex : `export PATH=/opt/apache-maven-3.5.2/bin:$PATH`

## Maven仓库

配置文件：*settings.xml*

### 本地仓库

```
<localRepository>C:\MyProgram\apache-maven-3.6.3\maven-repo</localRepository>
```

### 远程仓库（私服）

在公司局域网内

### 中央仓库

存放了基本所有的开源jar包

```xml
<mirror>
    <id>aliyunmaven</id>
    <mirrorOf>*</mirrorOf>
    <name>阿里云公共仓库</name>
    <url>https://maven.aliyun.com/repository/public</url>
</mirror>
```

## 标准目录结构

```
--src
----main
------java 核心代码部分
------resources 配置文件部分
------webapp (web项目)页面资源，js,css,图片等
----test
------java 测试代码部分
------resources 测试配置文件
```

