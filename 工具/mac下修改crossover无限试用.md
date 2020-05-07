https://www.jianshu.com/p/8f5d41a586bb



# 破解

### 修改无限试用时间方式破解

```tex
1.安装下载好的Crossover（当然是试用版了）

2.运行一次

弹出窗口提示试用天数 14天

选择试用－》打开－》然后退出Crossover

3.用mac下显示隐藏文件的工具显示所有文件
打开目录
/Users/你的用户名/Library/Preferences/com.codeweavers.CrossOver.plist
文件
修改
修改FirstRunDate,喜欢的话加个1000年吧
```



#### 或者mac设置无限试用，适用于所有程序

下面是方法操作

```bash
假如 安装程序为 Beyond Compare 

1 打开命令行终端，进入到安装目录里面的 Contents／Macos，这是可执行文件的存放目录

cd /Applications/Beyond\ Compare.app/Contents/MacOS/

2 把可执行文件改一下名，同时自己写一个脚本命名为这个可执行文件，以后每次打开此程序 都是通过这个脚本来打开的：

2.1 把可执行文件改名

mv BCompare BCompare.real

2.2 创建脚本
touch BCompare

2.3 给脚本可执行权限

chmod a+x BCompare

2.4 写脚本内容
#!/bin/bash

rm "/Users/$(whoami)/Library/Application Support/Beyond Compare/registry.dat"
```

