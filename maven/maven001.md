## maven的好处

完整的单元测试覆盖率报告

jar包依赖管理，不用复制到lib包。依赖包的依赖也自动下载

jar包冲突，版本调解

依赖管理、构建管理、工程管理、项目信息、模块化拆分管理

编译、打包、整合、发布，一键部署



make 不跨平台

ant 语法繁琐，依赖需要通过ivy

maven  工程管理工具

gradle 基于DSL语言进行构建管理，语法强大

## maven环境搭建

maven安装

安装jdk

安装maven

设置maven jvm参数，在系统环境变量设置 MAVEN_OPTS  设置堆内存

让 maven 自动构建 .m2文件夹，在cmd命令行 执行 mvn help:system



拷贝settings文件到.m2,作为全局配置文件

## 快速构建一个工程

mvn archetype:generate



mvn clean package



## maven命令

依赖下载，自动化执行编译、打包（构建，build）



## idea maven环境配置





手工命令打包，是mvn test package的时候才会去下载对应的依赖



## 通过idae构建一个简单的工程



## maven在实际项目中的使用



maven坐标



## maven依赖讲解

scope有哪些值

complie 默认值  编译 测试 运行的时候都有效

test

provided  编译测试有效，运行的时候不会使用，环境可能已经提供

runtime 编译测试 不用，运行的时候使用



### 传递性依赖

递归



### 依赖调解

谁离的近用谁

路径等长，谁先声明先用谁



<option></option> 如果有option属性，A依赖B,B依赖C(option)，则A不依赖C

## 依赖冲突

自动依赖调解，但是选的是错误的版本呢

依赖传递+依赖调解产生的

A-B-C.10

A-E-F-D-C-2.0

根据路径，最终使用C1.0，但是D报错了，使用了C2.0的新特性



mvn dependency:tree 看下整个项目的依赖路径树，看看依赖路径里，看看需要用哪个版本，手动指定特定的版本，排除掉1.0版本

<exclusions>

</exclusions>

这样系统就只使用2.0版本

## maven仓库

仓库布局：

​	



私服



镜像仓库，



## 安装私服



四种类型的仓库

group 仓库组，将各种仓库虚拟成一个组，配置group就可以使用各个仓库了

hosted仓库，上传公司自己的jar包

proxy 代理仓库，可以修改为阿里云仓库



## 安装企业级架构



修改代理仓库为阿里云代理仓库



18

配置镜像仓库，对任何jar的请求，直接走镜像

## nexus权限

编译  测试 打包  部署



权限认证

nexus默认有3个账户

admin   

deployment

anoymous 匿名账号。可以从私服下载依赖



## 20 组织机构配置账号



部署到私服 

mvn clean deploy  编译  单元测试 打包 先放到本地仓库  再放到配置的私服

snapshot版本上传会自带一个时间戳，每次改动都是一个新的，别人拉取的始终是最新的版本



mvn clean package 清理  编译 测试 打包

mvn clean install 清理 编译 测试 打包 安装到本地

mvn clean deploy