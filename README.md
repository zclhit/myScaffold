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

为了把react引入进来并且方便后续（可能的）前后端分离操作，同时提供更好的前后端本地调试体验，我们通过create-react-app的方式来完成，需要使用npm先安装create-react-app：

```npm install -g create-react-app```

安装成功后，我们通过create-react-app来初始化react app，在src目录下执行：

```create-react-app web```

## 配置frontend-maven-plugin

这里我们使用frontend-maven-plugin插件来实现项目打包时直接把build得到的前端文件放到Spring Boot使用的相关目录下，这样Spring Boot项目启动后可以直接访问到前端页面。

首先向pom.xml文件中添加frontend-maven-plugin插件：

```xml
<plugins>
  <plugin>
    <!-- https://mvnrepository.com/artifact/com.github.eirslett/frontend-maven-plugin -->
      <groupId>com.github.eirslett</groupId>
      <artifactId>frontend-maven-plugin</artifactId>
      <version>1.6</version>
  </plugin>
</plugins>
```

同时配置maven-plugin插件在执行阶段设置一系列行为去安装node和npm，执行npm install和npm run-script build。

```xml
<executions>
  <execution>
     <id>install node and npm</id>
     <goals>
       <goal>install-node-and-npm</goal>
     </goals>
     <configuration>
       <nodeVersion>v8.11.1</nodeVersion>
       <npmVersion>5.6.0</npmVersion>
       <nodeDownloadRoot>http://npm.taobao.org/mirrors/node/</nodeDownloadRoot>
       <npmDownloadRoot>http://npm.taobao.org/mirrors/npm/</npmDownloadRoot>
       </configuration>
   </execution>
   <execution>
     <id>npm install</id>
     <goals>
        <goal>npm</goal>
     </goals>
     <configuration>
        <arguments>install</arguments>
      </configuration>
   </execution>
 
   <execution>
     <id>npm run-script build</id>
     <goals>
       <goal>npm</goal>
     </goals>
     <configuration>
       <arguments>run-script build</arguments>
     </configuration>
    </execution>
 </executions>
 <configuration>
   <installDirectory>target</installDirectory>
   <workingDirectory>web</workingDirectory>
 </configuration>
```

其中配置选项中的installDirectory制定了node和npm的安装路径，在jar包中实现npm和node的安装可以很好解决在没有node和npm的机器上仍然能够运行，当然如果这台机器已经安装了node和npm，就会屏蔽掉全局设置而使用包内部的node和npm。

workingDirectory制定了前端项目package.json文件所在的路径，插件会自动在workdingDirectory路径下执行npm install和npm run-script build的命令。

可以看到用frontend-maven-plugin可以很好解决目标机器上没有npm和node的场景，但是在CI/CD场景下，这种做法会每次都重复安装node与npm，不适合持续集成，所以如果能保证打包机器安装了npm和node，可以去掉install node and npm这一步。

## 配置webpack

为了能够使用webpack，我们在package.json中增加以下的devDependencies:

```json
"clean-webpack-plugin": "^3.0.0",
"webpack": "^4.42.0",
"webpack-cli": "^3.3.11",
"webpack-dev-server": "^3.10.3",
"webpack-merge": "^4.2.2"
```

添加完成之后，记得去web目录下执行```npm install```安装最新的依赖。

之后在web目录下创建build目录，在build目录中进行webpack的相关配置。这里针对dev环境和prod环境进行不同的配置，由于dev环境下我们会直接使用npm start暴漏3000端口并转发至localhost:8080端口进行本地前后端联调，所以需要在本地进行devServcer的配置：

```javascript
devServer: {
        port: '3000',
        contentBase: path.join(__dirname, '../dist'),
        compress: true,
        hot: true,
        proxy: {
            '/*': {
                target: 'http://localhost:8080'
            }
        }
    }
```

同时，在package.json中指定不同的script来实现不同的行为：

```json
"scripts": {
    "dev": "webpack-dev-server --config build/webpack.dev.conf.js",
    "start": "npm run dev",
    "build": "webpack --config build/webpack.prod.conf.js"
  },
```

比如，我们经常通过npm start来启动一个本地服务，那么就可以让他执行dev，然后走webpack.dev.con.js中的设置，启动一个本地服务，proxy到本地的8080端口。

而在生产环境打包的过程中我们可以能需要npm run build，那么我们可以让它走webpack.prod.conf.js中的逻辑，把相应的包打包到Spring Boot的template或者static路径下，这样Spring Boot在启动之后就可以直接访问通过React编写好的页面。

## 后记

我把sprint-boot-starter-web之外增加了spring-boot-starter-thymeleaf，因为我认为thymeleaf提供更多优秀的功能。大家可以在使用的过程中自由选择。

