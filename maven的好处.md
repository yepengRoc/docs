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

