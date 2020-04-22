# myScaffold
A scaffold based on Spring Boot and React, integrate them together and deploy fast.

# Spring Boot与React集成在同一个项目实现脚手架

## 前言

最近在考虑重新搭建自己的项目开发脚手架，我给这套脚手架的定义的期望是：足够高的集成度与足够简洁，满足快速上手开发与快速产出的要求。考虑到这些期望，我选择了Spring Boot作为后端实现框架，而前端，作为2019年stack over flow most loved framework的票王，我选择了React.js。

## 环境搭建

这个项目会依赖于Jdk1.8，node 8和npm 6进行展示，在此之前请确保你已经安装了上述依赖到当前的开发环境。

## 创建Spring Boot with web

通过IntelliJ IDEA自带的Spring Boot生成器生成web项目，File - New -  project... - Spring Initializer (NEXT)，设置好坐标名、项目名称，打包方式，Java版本，项目描述等信息。

![image-20200422103217150](/Users/changle.zhang/Library/Application Support/typora-user-images/image-20200422103217150.png)

在选择项目依赖时，勾选Spring Web，这时将会自动将spring-boot-start-web依赖添加到项目中。spring-boot-start-web提供了开发一个Spring boot web所需要的绝大多数依赖，可以省下很多时间去做一些自己的设置和增加额外的依赖。

![image-20200422105231907](/Users/changle.zhang/Library/Application Support/typora-user-images/image-20200422105231907.png)

可以看到，初始化之后的文件结构如图所示，所有的业务逻辑代码都在src/main/java目录下，所有的配置信息与静态文件都在resource下，所有的测试文件都在src/main/test目录下，由于我选择的是maven作为项目管理工具，所以mvnw, mvnw.cmd和pom.xml会出现在这个目录下。

![image-20200422111523521](/Users/changle.zhang/Library/Application Support/typora-user-images/image-20200422111523521.png)

至此，一个基本的Spring Boot with Web就完成了。

## 把React引入进来



