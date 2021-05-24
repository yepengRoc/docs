### 系统gcc版本过低需要安装新的gcc



gcc 下载地址

http://ftp.gnu.org/gnu/gcc

这里以 7.1版本进行安装

1. 可以windows上直接下载，然后上传到linux服务器

2. 可以直接在linux上使用 wget 进行下载

   wget http://ftp.gnu.org/gnu/gcc/gcc-7.1.0/gcc-7.1.0.tar.gz

在linux进行解压

tar -xvf gcc-7.1.0.tar.gz

cd  gcc-7.1.0  

```shell
./contrib/download_prerequisites   
cd ..   #返回上级目录
mkdir build_gcc_7.1.0
cd build_gcc_7.1.0
../gcc-7.1.0/configure --enable-checking=release --enable-languages=c,c++ --disable-multilib
make -j8 #这里需要说明下j job的意思，后面的数字8 标识起8个job进行处理。因为我这里的服务器是4核8线程，所以我这里用的是8.可根据资金的服务器情况合理设置
make install

#进行查看  默认是装在 /user/local/bin
ls /usr/local/bin | grep gcc

```

这个时候查看 gcc 和g++ 已经变为 7.1.0

如果没有变过来的话，需要执行以下操作

```shell
/usr/sbin/update-alternatives --install  /usr/bin/gcc gcc /usr/local/bin/x86_64-pc-linux-gnu-gcc-7.1.0 40 # 40标识的是一个优先级，数字越大，优先级越高
gcc --version      #查看版本
/usr/sbin/update-alternatives --install /usr/bin/g++ g++ /usr/local/bin/g++ 40
g++ --version     #查看版本
```



```shell
cp build_gcc_7.1.0/x86_64-pc-linux-gnu/libstdc++-v3/src/.libs/libstdc++.so.6.0.23  /usr/lib64/
#删除原来的libstdc++.so.6
rm libstdc++.so.6
#重新建立软连接
ln libstdc++.so.6.0.23 libstdc++.so.6
ln –snf  libstdc++.so.6.0.23  libstdc++.so.6  #更新
#执行以下命令查看
strings /usr/lib64/libstdc++.so.6 | grep GLIBCXX


```
# glibc 需要升级
**直接通过修改软链的方式不行**

```shell
strings /lib64/libc.so.6 | grep GLIBC ##查看 glibc 有哪些版本
rm -rf /lib64/libc.so.6  # 删除libc软链
ln -s /lib64/libc-2.15.so  lib64/libc.so.6 # 新建软链。掉坑里了。linux系统依赖 libc，删了之后不能正常运作了

sudo ln –snf /lib64/libc-2.18.so /lib64/libc.so.6 # 更新软链也不行

##删除软链之后的恢复方法，需要root权限。前提是没有退出命令行，退出就完蛋了，只能重装，都登录不了

LD_PRELOAD=/lib64/libc-2.12.so ln -s /lib64/libc-2.12.so  /lib64/libc.so.6


##这个方式不好使 
LD_PRELOAD=/lib64/libc-2.12.so rm libc.so.6
LD_PRELOAD=/lib64/libc-2.18.so ln -s /lib64/libc-2.18.so libc.so.6

LD_PRELOAD=/lib64/libc-2.18.so ln -s /lib64/libc-2.18.so libc.so.6


LD_PRELOAD=/data/libc-2.18-backups.so rm /lib64/libc.so.6
LD_PRELOAD=/data/libc-2.18-backups.so ln -s /data/libc-2.18-backups.so /lib64/libc.so.6
libc-2.18-backups.so

LD_PRELOAD=/lib64/libc-2.18.so rm /lib64/libc.so.6
LD_PRELOAD=/lib64/libc-2.18.so ln -s /lib64/libc-2.18.so /lib64/libc.so.6
```

**需要从新下载新的glib包来解决glic版本升级问题**

```shell
下载地址http://ftp.gnu.org/gnu/glibc/

wget http://ftp.gnu.org/gnu/glibc/glibc-2.18.tar.gz

tar -xvf glibc-2.18.tar.gz

mkdir build_glibc-2.18

cd build_glibc-2.18

../glibc-2.18/configure  --prefix=/usr --disable-profile --enable-add-ons --with-headers=/usr/include --with-binutils=/usr/bin

make -j8

make install
```



### 安装rz sz

yum install lrzsz



修改java 环境

/usr/java/jdk1.8.0_92



root密码：

uat  root密码：  Chunb0@2o14

prod  root密码： Chunb0@2o15888



### 软链的相关操作

**创建软链接**

ln  -s [源文件或目录] [目标文件或目录]

例如：

当前路径创建test 引向/var/www/test 文件夹 

*ln –s  /var/www/test test*

创建/var/test 引向/var/www/test 文件夹 

*ln –s  /var/www/test  /var/test* 

**删除软链接**

和删除普通的文件是一眼的，删除都是使用rm来进行操作

 rm –rf 软链接名称（请注意不要在后面加”/”，rm –rf 后面加不加”/” 的区别，可自行去百度下啊）

例如：

删除test

*rm –rf test*

**修改软链接**

ln –snf  [新的源文件或目录] [目标文件或目录]

这将会修改原有的链接地址为新的地址

例如：

创建一个软链接

*ln –s  /var/www/test  /var/test*

修改指向的新路径

*ln –snf  /var/www/test1  /var/test*



如果您想更深入的了解，可以ln –help 查看详细。