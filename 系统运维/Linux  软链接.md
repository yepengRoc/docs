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

 rm –rf 软链接名称（请注意不要在后面加”/”，rm –rf 后面加不加”/” 的区别，可自行去百度）

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



更多内容，可以ln –help 查看详细。