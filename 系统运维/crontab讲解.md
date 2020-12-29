https://www.jianshu.com/p/838db0269fd0



### crontab配置文件

- 其一：`/var/spool/cron/`
   该目录下存放的是每个用户（包括root）的crontab任务，文件名以用户名命名

  将对应要执行的命令添加进来之后就生效了，通过 crontab -l进行查看

- 其二：`/etc/cron.d/`
   这个目录用来存放任何要执行的crontab文件或脚本。

###  cat /etc/crontab   查看crontab 表达式各个字段的意思

```bas
SHELL=/bin/bash
PATH=/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=root
HOME=/

# For details see man 4 crontabs

# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name command to be executed
```

crontab时间说明

minute：代表一小时内的第几分，范围 0-59。
 hour：代表一天中的第几小时，范围 0-23。
 mday：代表一个月中的第几天，范围 1-31。
 month：代表一年中第几个月，范围 1-12。
 wday：代表星期几，范围 0-7 (0及7都是星期天)。
 who：要使用什么身份执行该指令，当您使用 crontab -e 时，不必加此字段。
 command：所要执行的指令。



### crontab服务状态

```bash
sudo service crond start     #启动服务
sudo service crond stop      #关闭服务
sudo service crond restart   #重启服务
sudo service crond reload    #重新载入配置
sudo service crond status    #查看服务状态
```

### crontab命令

 重新指定crontab定时任务列表文件

```shell
crontab $filepath
```

查看crontab定时任务

```undefined
crontab -l
```

编辑定时任务【删除-添加-修改】

```undefined
crontab -e
```

### 添加定时任务【推荐】

 第一步 : 编辑任务脚本【分目录存放】【ex: backup.sh】
 第二步 : 编辑定时文件【命名规则:backup.cron】
 第三步 : crontab命令添加到系统`crontab backup.cron`
 第四步 : 查看crontab列表 `crontab -l`