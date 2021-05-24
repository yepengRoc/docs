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

修改系统gcc软链指向。如果有别人编译好的，直接拷贝到/usr/lib64 目录下修改，软链接即可

```shell
cp build_gcc_7.1.0/x86_64-pc-linux-gnu/libstdc++-v3/src/.libs/libstdc++.so.6.0.23  /usr/lib64/
#删除原来的libstdc++.so.6
rm libstdc++.so.6
#重新建立软连接
ln libstdc++.so.6.0.23 libstdc++.so.6
# 或者直接更新软链接 。我这里不好使，上面的删除重检可以
ln –snf  libstdc++.so.6.0.23  libstdc++.so.6  #更新软链
#执行以下命令查看
strings /usr/lib64/libstdc++.so.6 | grep GLIBCXX


```




