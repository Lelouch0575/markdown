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

## IDE集成maven

### IDEA

File--settings--Build--Build Tools--Maven，选择Maven路径和配置文件路径和仓库路径。

#### 创建Web项目

创建maven项目，勾选create from archetype，选择maven-archetype-webapp，补全缺失的文件夹，在文件夹上右键--Mark Directory as进行标记。（也可在File--Project Structure--modules下设置）

***

# Gradle

## 环境变量

`GRADLE_HOME  %GRADLE_HOME%\bin`

## 使用Maven仓库

创建环境变量`GRADLE_USER_HOME`指定仓库路径，修改`build.gradle`

```
repositories {
	//先让gradle从本地仓库找,找不到再从下面的mavenCentral()中央仓库去找jar包
    mavenLocal()
    mavenCentral()
}
```

