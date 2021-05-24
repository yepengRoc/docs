java 开发 使用google Or-Tools 

官网地址：https://developers.google.cn/optimization/



## 搭建本地java开发环境：

在maven中加入依赖的包。

如果查找依赖的包



安装Or-Tools,详见官网首页

![image-20210203164541302](image/image-20210203164541302.png)

进去之后

![image-20210203164610768](image/image-20210203164610768.png)

我选择的是windos版本

![image-20210203164709858](image/image-20210203164709858.png)

下载解压之后是这样

![image-20210203164748383](image/image-20210203164748383.png)

这里使用的最新版本是 8.1.8487，但是在公网maven仓库中并没有搜到，需要把ortools-java-8.1.8487.jar 和ortools-win32-x86-64-8.1.8487.jar手动上传公司的私服

然后再pom文件中加入依赖。这里说明下理论上只在pom中加入 ortools-java-8.1.8487.jar 的依赖记录即可，因为这个jar包依赖了 ortools-win32-x86-64-8.1.8487.jar。

后来我发现不好使，我把ortools-java-8.1.8487.jar 依赖的jar都放入了进来，解压ortools-java-8.1.8487.jar 下有一个pom文件，里面有这个jar依赖的其它 jar配置.

因为or-tools运行，需要使用到系统的c库和c++库，加载系统c库和c++库是通过ortools-win32-x86-64 实现的，win和linux的实现不一样，可找到![image-20210203165612272](image/image-20210203165612272.png)进行下载，然后把![image-20210203165642230](image/image-20210203165642230.png)上传公司私服，直接在pom文件中把linux需要使用的jar也加入进来，这样部署到linux服务器的时候也可以正常使用。

完整pom配置

```xml
<dependency>
			<groupId>com.google.ortools</groupId>
			<artifactId>ortools-java</artifactId>
			<version>8.1.8487</version>
			<scope>compile</scope>
		</dependency>
		<dependency>
			<groupId>com.google.ortools</groupId>
			<artifactId>ortools-win32-x86-64</artifactId>
			<version>8.1.8487</version>
			<scope>runtime</scope>
		</dependency>
		<dependency>
			<groupId>com.google.ortools</groupId>
			<artifactId>ortools-linux-x86-64</artifactId>
			<version>8.1.8487</version>
			<scope>runtime</scope>
		</dependency>

		<dependency>
			<groupId>net.java.dev.jna</groupId>
			<artifactId>jna-platform</artifactId>
			<version>5.5.0</version>
		</dependency>
		<dependency>
			<groupId>com.google.protobuf</groupId>
			<artifactId>protobuf-java</artifactId>
			<version>3.14.0</version>
		</dependency>
```

## 安装or-tools依赖的c和c++库

运行or-toos实例程序的时候提示无对应的库 

### windows上安装

下载VC_redist.x64.exe  双击安装即可

### Linus CentOs上安装

运行程序的时候，提示 GLIBCXX_3.4.14' not found  和GLIBC_2.14 not found

GLIBCXX_3.4.14'  是c++库

GLIBC_2.14 是c库

经验证 c++ 库如果有别人编译好的高版本库，直接在 /usr/lib64 目录下从新建立软链接即可

/lib64/下的 c库则不行，这里比较坑爹，按照网上的步骤 找了一个高版本的库，先删除，再建立软链接，删除后发现整个系统都操作不了了，

自己又退出了，导致整个服务器都登录不了，后来只有把服务器重做了。这里要说明下，所有的系统操作都依赖这个c库，千万不要删除，千万不要删除，千万不要删除！！！ 我在这里验证c库，即便把高版本的库改名字覆盖低版本的库，也不行，必须编译安装高版本的c库。