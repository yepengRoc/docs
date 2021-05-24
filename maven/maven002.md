## 自动执行管理任务

在system 菜单

tasks 下创建定时任务

## 22maven生命周期和plugin





clean 

- pre-clean
- clean
- post-clean

default

- 

site  生成工程的一些文档

- pre-site  准备工作
- site 生成site
- post-site 生成site的一些工作
- site-deploy 部署到远程



mvn 命令后可以跟任何phase, maven默认会执行 当前phase和 这个phase之前的 phase

​	每个phase  会绑定一些plugin  goal



mvn deploy:deploy-file

mvn dependency:tree 

不执行任何phase

## 23 plugin





测试插件 cobertura,计算测试覆盖率

jetty 启动插件，不用打war包到tomcat



如何配置插件 



插件配置语法



- 插件和goal



- 

每一个excution就代表一次执行

<id></id>

<phase></phase> 指定哪个阶段

<goals><goal></goal></goals> 指定哪些goal

可以指定多个<excution>



插件里还可以配置configration



## 24授权模块演示



## 25请假



id  employee_id days  表字段



## 26基于聚合，进行构建

多独立工程互相依赖

每次改动，需要测试功能通过



通过父模块，聚合所有模块，来管理所有模块

问题：子模块都部署到私服了，父模块没有部署成功，导致其它项目对这个工程依赖，只能正常下载子模块，无法下载到父模块



## 27基于继承统一 各模块版本



所有版本声明在父工程中，子工程 使用parent 元素声明继承，强制继承父工程所有的插件和依赖。

其实子工程不是所有的都需要，可能只需要部分

推荐做法是，在父工程中 使用 depdepencyManagement  pluginManagement ，不要求子工程强制依赖

，如果需要只需要在子工程中声明 aritifactId  groupId即可，不用声明版本号



使用方式：需要公共持有的就不放 dependencyManagement中，需要根据各个工程自己选择的就放dependencyManagement中



插件同样可以这样处理。只有自己工程需要的插件，定义出来



## 28企业场景实战

jar包版本号

如果需要升级版本号，需要改各个工程



在父pom文件中，定义统一的版本号

## 29实战场景



对于公共jar包的管理，如果公共jar包又依赖其他jar包，自己的工程也依赖了其它jar包，版本不一致，导致工程最后使用的是自己工程内的jar包，公共jar报错了

建立父pom，通过dependencyManage 管理依赖，

公共jar包，只依赖自己的父pom的

## 30oaweb服务完善

创建一个web工程，依赖其它3个工程



## 31surefire 测试覆盖率报告



持续集成，自动化集成测试

jendkins handson travis

持续集成，每个工程师，让持续集成服务器跑各个工程师的 测试用例



surefire插件



方法覆盖率

行覆盖率

讲的好



## 32实战

jetty插件

可以实现，不用打包 部署到tomcat 直接启动测试



settting文件中的pluginGroups 作用

配置插件所有和插件的映射关系，这样输入命令的时候，可以配置插件缩写



## 33 插件 cargo 实现自动化部署

把本地的war包 送到远程tomcat服务上



## 34资源适配，加profile 适配各个发布环境



通过打包的时候 -P各个环境，

大型工程里的配置特别多，一般不放profiles 里，而是建立各个对应的目录，

<properties> 去掉

使用 build  resource 引入对应的目录



## 35版本管理版本控制



版本管理 version 控制粒度粗，整个开发阶段都是一个sanpshot版本



版本控制 git  记录每一次改动，都是一个版本

## 36版本管理

版本号的 意义



## 37项目工程骨架

archetype  类似于一个项目模板，生成一些基础信息，然后基于这些基础信息 进行增删改，然后进入开发



为公司创建统一的工程骨架

自定义出一个archetype，通用模板

讲的很实用

## 38

scm 

ciManagement



shade插件，依赖的jar 打包到当前包中

jacoco 插件，静态代码检测

## 39总结

