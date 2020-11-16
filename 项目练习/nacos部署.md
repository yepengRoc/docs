git下载源码，然后

mvn -Prelease-nacos  -Dmaven.test.skip=true  clean install -U

ls -al distribution/target/

cd distribution/target/nacos-servier-*/nacos/bin



tar.gz 或者 .zip 上传到三台机器

重命名 cluster.conf.example

tar -zxvf  *.tar.gz

cd 解压的目录

mv cluster.conf.examle cluster.conf

配置三台机器的地址和端口，默认使用的是derby 数据库，可以配置mysql数据库，

使用nacos.mysql 建立对应需要的表，然后配置对应的数据源

数据源修改在 application.properties 文件

 spring.datasource.platform=mysql

db.num=1

db.url.0=

db.user=

db.password=



分别进入集群的每台服务器的bin目录，执行startup.sh,检查logs目录下的start.out启动日志。



访问任何一个节点的 8848端口的 /nacos 地址，进入 nacos的控制台，可以查看集群的具体情况





