---
title: springboot 引用第三方jar包以及打包
date: 2018-01-09 16:33:11
tags: 
  - java
  - maven 
  - springboot
  - jar
---

项目springboot架构，maven打包需要war包

由于业务需要使用马云爸爸的jar包，而他的jar包是根据用户权限及时生成的，所以中央maven库没有。

百度谷歌一番，本地使用没有问题，引用代码如下：

```xml
<dependency>
  <groupId>org.miemie.analyzer</groupId>  <!-- 随意-->
  <artifactId>tbk</artifactId>   <!-- 随意-->
  <version>1.0</version>  <!-- 随意-->
  <scope>system</scope>  <!-- /system 表示不读取本地仓库，读取指定系统文件，默认为compile（编译测试打包都会放进去）-->
  <systemPath>${project.basedir}/lib/xxxx.jar <!-- src同级目录新建一个lib文件夹，将需要的jar包扔进去-->
  </systemPath>
</dependency>

```



> 问题主要在打包，如果正常打包部署，服务器会报nosuchclass错误，因为没打进去啊,那么就要解决第三方jar包无法打入的问题




<!--more-->

### 打包大致有几种解决方法：

1. 本地安装jar包到maven仓库，直接引用，不用像上面代码一样写范围和路径了。简单粗暴，打jar/war包都可以，但是缺点很明显，每台电脑都需要安装一遍。

2. 统一配置 (包括打jar和war包)，这样不用每台pull项目的机器都安装jar包
+ 打jar包，在pom文件中的build标签下加上如下的配置
```xml
<build>  
    <plugins>  
        <plugin>  
            <groupId>org.springframework.boot</groupId>  
            <artifactId>spring-boot-maven-plugin</artifactId>  
        </plugin>  
    </plugins>  
    <resources>  
        <resource><!-- 将lib目录下的jar包拷贝到BOOT-INF/lib/目录下 -->
            <directory>${project.basedir}/lib</directory>  
            <targetPath>BOOT-INF/lib/</targetPath>  
            <includes>  
                <include>**/*.jar</include>  
            </includes>  
        </resource>  
        <resource><!-- 如果不加上这个配置，不然src/main/resource目录下的配置文件就不会打到jar包下去了  -->
            <directory>src/main/resources</directory>  
            <targetPath>BOOT-INF/classes/</targetPath>  
        </resource>  
    </resources>  
</build>  

  ```

+ 打war包，要用到maven-war-plugin 插件
```xml
<plugin>  
  <groupId>org.apache.maven.plugins</groupId>  
  <artifactId>maven-war-plugin</artifactId>  
  <configuration>  
    <webResources>  
      <resource>  
        <directory>${project.basedir}/lib</directory>
        <targetPath>WEB-INF/lib/</targetPath>  
        <includes>  
          <!-- 把所有lib下的jar包打进WEB-INF/lib里面-->
          <include>**/*.jar</include>  
        </includes>  
      </resource>  
    </webResources>  
  </configuration>  
</plugin>

```


> 至此，应该都可以愉快的玩耍了，把你的jar/war包丢到服务器吧。


