# glibc 需要升级

**直接通过修改软链的方式不行**

```shell
strings /lib64/libc.so.6 | grep GLIBC ##查看 glibc 有哪些版本
rm -rf /lib64/libc.so.6  # 删除libc软链
ln -s /lib64/libc-2.15.so  lib64/libc.so.6 # 新建软链。掉坑里了。linux系统依赖 libc，删了之后不能正常运作了

sudo ln –snf /lib64/libc-2.18.so /lib64/libc.so.6 # 更新软链也不行

##删除软链之后的恢复方法，需要root权限。前提是没有退出命令行，退出就完蛋了，只能重装，都登录不了

LD_PRELOAD=/lib64/libc-2.12.so ln -s /lib64/libc-2.12.so  /lib64/libc.so.6


##如果误删除了 libc.so.6或者 让 libc.so.6指向了不能用的c库，我这里进行了覆盖操作，导致不能用了, 可通过以下方式恢复
LD_PRELOAD=/lib64/libc-2.12.so rm libc.so.6 #如果是覆盖操作,需要先删除软链
LD_PRELOAD=/lib64/libc-2.18.so ln -s /lib64/libc-2.18.so libc.so.6 # 删除后，利用原来自带的可用版本 重建软链


LD_PRELOAD=/lib64/libc-2.18.so rm /lib64/libc.so.6
LD_PRELOAD=/lib64/libc-2.18.so ln -s /lib64/libc-2.18.so /lib64/libc.so.6
```

**需要从新下载新的glib包来解决glic版本升级问题**

```shell
下载地址http://ftp.gnu.org/gnu/glibc/

wget http://ftp.gnu.org/gnu/glibc/glibc-2.18.tar.gz  #下载glib 包

tar -xvf glibc-2.18.tar.gz #解压

mkdir build_glibc-2.18 #新建一个用来编译glibc的目录

cd build_glibc-2.18 # 进入编译目录

../glibc-2.18/configure  --prefix=/usr --disable-profile --enable-add-ons --with-headers=/usr/include --with-binutils=/usr/bin #配置环境

make -j8 #编译 8指的是起几个任务进行编译，根据自己服务器的核心数进行合理设置，我的服务器是4核8线程，我设置的为8，线上环境可设置小点  4即可，免得影响了其它正在跑的应用

make install #安装

strings /lib64/libc.so.6 | grep GLIBC ##查看 glibc 有哪些版本。发现安装已生效
```

