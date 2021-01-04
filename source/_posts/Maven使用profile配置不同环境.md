---
title:  Maven使用profile配置不同环境
tags:
  - Maven 
categories:
  - Maven
comments: true
date: 2021-1-4 20:00:00

---

Maven 中有一个概念叫做：`profile`，它的诞生主要是为了解决不同环境所需的不同变量、配置等问题。有了 profile，可以根据激活的条件，启动不同条件下的配置信息。profile 是可以有多个的，也可以同时激活多个 profile，方便自由组合。profile 一般可以在三个地方：settings.xml，pom.xml，profiles.xml（这个不常用）

<!--more-->

## 配置分为以下几个步骤：

1. 配置profiles节点（pom.xml）

2. 配置build节点（pom.xml)--如果不配置该节点则无法找到profile中的properties属性值，并且配置后超链接才有效

3. 在xml或properties中使用

4. 使用



### 1、配置profiles节点（pom.xml）

```maven
<profiles>
    <profile>
        <!-- 开发环境 -->
        <id>dev</id>
        <properties>
            <!-- 一些自定义配置，如果选中改环境，则可以使用${env}或${profile}占位符使用，但必须配置build才会生效 -->
            <env>dev</env>
            <profile>dev</profile>
        </properties>
        <activation>
            <!-- 默认激活该profile节点-->
            <activeByDefault>true</activeByDefault>
        </activation>
    </profile>

    <profile>
        <!-- 测试环境 -->
        <id>test</id>
        <properties>
            <env>test</env>
            <profile>test</profile>
        </properties>
    </profile>

    <profile>
        <!-- 发布环境 -->
        <id>prod</id>
        <properties>
            <env>prod</env>
            <profile>prod</profile>
        </properties>
    </profile>
</profiles>
```



### 2、配置build节点（pom.xml)

如果不配置该节点则无法找到profile中的properties属性值，并且配置后超链接才有效

只需要在pom文件中配置一个就可以，下面三个做的事情一样，只是细节不同，具体怎么配，要根据项目结构来决定

格式一：

```maven
<build>
    <finalName>${project.artifactId}-${project.version}</finalName>
    <!--资源文件处理-->
    <resources>
        <resource>
            <directory>src/main/resources</directory>
            <!--是否开启属性替换过滤-->
            <filtering>true</filtering>
            <includes>
                <include>**/*.*</include>
            </includes>
            <!--打包时先排除掉四个文件夹-->
            <excludes>
                <exclude>dev/*</exclude>
                <exclude>test/*</exclude>
                <exclude>uat/*</exclude>
                <exclude>pro/*</exclude>
            </excludes>
        </resource>
        <resource>
            <directory>src/main/resources/${env}</directory>
            <includes>
                <include>**/*.*</include>
            </includes>
        </resource>
    </resources>
<build>
```



格式二：

```maven
<resources>
    <resource>
        <directory>src/main/resources</directory>
        <excludes>
            <exclude>**/*.yml</exclude>
        </excludes>
    </resource>
    <resource>
        <directory>src/main/java</directory>
        <includes>
            <include>**/*.xml</include>
        </includes>
        <filtering>false</filtering>
    </resource>
    <resource>
    <directory>src/main/resources</directory>
        <filtering>true</filtering>
        <includes>
            <include>application.yml</include>
            <include>application-${profile}.yml</include>
            <include>**/*.xml</include>
        </includes>
    </resource>
</resources>
```



格式三：

```maven
<!--资源文件处理-->
<resources>
    <resource>
        <directory>src/main/resources</directory>
        <!--先排除所有的配置文件-->
        <excludes>
            <exclude>application*.yml</exclude>
        </excludes>
    </resource>
    <resource>
        <directory>src/main/resources</directory>
        <!--引入所需环境的配置文件-->
        <!--是否对 ${属性名} 进行替换-->
        <filtering>true</filtering>
        <includes>
            <include>application.properties</include>
            <include>application.yml</include>
            <include>application-${profile}.yml</include>
        </includes>
    </resource>
</resources>
```



### 3、在xml或properties中使用

在`application.yml`中配置

```java
spring:
  profiles:
    active: ${profile}
```



在`application-XXX.yml`中配置

```java
server:
  port: 8080
  servlet:
    context-path: /proXXX
```

也可以使用在配置profiles节点时**，**properties中自定义配置参数，如果选中该环境，则可以使用${属性名}占位，例如：${env}或${profile}，当`<filtering>true</filtering>`时会自动过滤，必须配置build才会生效 

### 4、使用

![](https://cdn.jsdelivr.net/gh/javahub-yuan/forBlogImages@master/img/profiles.png)

![](https://cdn.jsdelivr.net/gh/javahub-yuan/forBlogImages@master/img/package.png)

对jar包反编译，例如选择dev环境，jar包中就不会有其他环境的配置文件了，且配置文件中${占位符}已经被替换

![](https://cdn.jsdelivr.net/gh/javahub-yuan/forBlogImages@master/img/result1.png)

![](https://cdn.jsdelivr.net/gh/javahub-yuan/forBlogImages@master/img/result.png)

**参考**

https://www.cnblogs.com/gossip/p/6072601.html

https://blog.csdn.net/qq_43377237/article/details/103869813?utm_medium=distribute.pc_relevant.none-task-blog-baidujs_title-7&spm=1001.2101.3001.4242

https://blog.csdn.net/qq_33626745/article/details/52857110?utm_medium=distribute.pc_relevant.none-task-blog-searchFromBaidu-1.not_use_machine_learn_pai&depth_1-utm_source=distribute.pc_relevant.none-task-blog-searchFromBaidu-1.not_use_machine_learn_pai

https://blog.csdn.net/xiaojiesu/article/details/51866303?utm_medium=distribute.pc_relevant_t0.none-task-blog-searchFromBaidu-1.not_use_machine_learn_pai&depth_1-utm_source=distribute.pc_relevant_t0.none-task-blog-searchFromBaidu-1.not_use_machine_learn_pai

https://blog.csdn.net/u010010606/article/details/79727438

https://youmeek.gitbooks.io/intellij-idea-tutorial/content/maven-skill-introduce.html

