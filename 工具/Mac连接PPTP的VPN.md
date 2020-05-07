# Mac 10.14使用pppd連接PPTP的VPN（官方組件）

# Mac系統版本

> Mac OS版本：10.14.4 Mojave

我們都知道MAC在很早之前，VPN連接中已經不能添加PPTP的協議連接了（可能是安全性考慮吧），也就是說我們不能在`設置`-`網絡`中直接配置PPTP的VPN！這應該是很多朋友不敢更新MAC的原因（特別是使用第三方客戶端都連不上的人，畢竟工作相關）
但是，我說不能配置PPTP是指UI裏面的，實際上還是能夠通過配置`pppd`的方式連接PPTP的。
並非Shimo等第三方客戶端。

# 一、起因

發現這個方法是因爲某次我需要連接某貓星的VPN拉代碼（SVN倉），但對方只給我提供了PPTP協議，還不支持L2TP，也就是說通過`網絡`設置不了VPN。然後我按照別人說的下載了`Shimo`（這個是收費的軟件），更慘烈的是Shimo居然連接不上，錯誤原因提示還少得可憐。收費軟件很多，試過幾個都不行。

# 二、發現免費的解決方案

後面通過一輪的查找資料發現，Mac自帶的VPN配置是通過調用`pppd`組件進行撥號通信的，然後我還是發現了在終端中使用`pppd`是能夠建立PPTP的，蘋果官方只是刪除了UI的才做入口而已。`pppd`連接`PPTP`的功能還是存在的。
前提是你得配置對應的`撥號配置文件`。

# 三、普及下资料

首先，得告诉大家Mac的VPN配置文件都放在：

> /etc/ppp/peers/

這個目錄下。

# 四、开始配置

## 1. 建立PPTP拨号配置

打开終端，编辑配置，輸入：

```bash
sudo vim /etc/ppp/peers/inner
```

划重点：里面的`文件名`就是连接名称！！！**
所以我上面輸入`inner`就是`連接的名稱`，後面進行撥號的時候會用到！起名`inner`表示內部的意思。

## 2. 配置文件內容

然後在`vim`中輸入如下配置信息：

```bash
plugin PPTP.ppp
noauth
remoteaddress "----host----"
user "----username----"
password "----password----"
redialcount 1
redialtimer 5
idle 1800
# mru 1368
# mtu 1368
receive-all
novj 0:0
ipcp-accept-local
ipcp-accept-remote
refuse-eap
refuse-pap
refuse-chap-md5
hide-password
mppe-stateless
mppe-128
# require-mppe-128
looplocal
nodetach
# ms-dns 8.8.8.8
usepeerdns
# ipparam gwvpn
defaultroute
debug
```

> 其中：
> remoteaddress：雙引號內寫VPN服務器訪問地址
> user：用戶名
> password：密碼

替換成你的撥號賬號吧！

其他的加了`#`表示註釋了，不管它，留着吧，可能以後用得上~~~

## 3. 執行PPTP連接和終止連接

**在执行拨号之前，必須先聲明一下：**

> pppd撥號不是守護進程方式運行的，會一直搶佔終端的線程，並且不能通過`Ctrl + C`的方式終止。
> 需要使用`pkill`來殺掉整個進程！

（1）**在`終端`進行PPTP撥號：**

```bash
sudo pppd call inner
```

上面的就是表示`pppd`使用`inner`配置進行撥號，然後會彈一堆的log出來，這是因爲用了debug模式。
連接之後可以使用`ifconfig`（新終端窗口或tmux）查看IP地址是否撥號成功（log裏面也會有內容的）

（2）**終止PPTP連接：**
這個時候你當前的終端界面是在pppd進程中的，沒有退出，這個終端是不能進行操作的！(Ctrl+C也沒用的)
這個時候**新建`終端`窗口**，輸入：

```bash
sudo pkill pppd
```

這時候`pppd`進程會被殺掉，撥號的那個終端窗口會退出pppd進程的操作。

------

------

# 五、最後：使用`alias`進行偷懶：

每次輸入：

> sudo pppd call …
> sudo pkill pppd
> 手都軟了。

我們可以使用`alias`來簡化這些操作：
打開終端，輸入：

```bash
vim ~/.bash_profile
```

在最後面加入：

```bash
alias vpn-on='sudo pppd call inner'
alias vpn-off='sudo pkill pppd'
```

保存後刷新下：

```bash
source ~/.bash_profile
```

這時候就可以通過：

```bash
vpn-on
vpn-off
```

來控制VPN開關。不過還是記住，這個要分別在兩個終端窗口中才能操作。推薦使用`tmux`終端複用這個組件。

> emmm…使用終端連接PPTP還是有點geek味道的

------

此文同時在簡書發佈：https://www.jianshu.com/p/c69f38f95217
此文同時在CSDN發佈：https://blog.csdn.net/nthack5730/article/details/89774908
转载要加原文链接！謝謝支持！