# 树莓派网络与更新配置

## 有线网络：

自动获取 IP：树莓派默认有线网卡是使能的，只需将网线插入树莓派网卡，即可自动获得 IP(要求在局域网内)。

手动设定 IP：如果是电脑与树莓派直连，不能自动获得IP，可以使用：ifconfig eth0 192.168.1.123 设定 ip(下次重启就没了)。

设置静态 IP：如果担心在同网络情况下 ip 或者不固定，可以讲电脑设置为静态 ip，方法如下：

在终端中打开一下文件：

`sudo vi  /etc/network/interfaces`

(需要学习 VI 基本操作，树莓派自带的 vi 不是很好用，请先参考下面文档更新一下 vi）

```
auto lo
iface lo inet loopback
iface eth0 inet dhcp
```

将以上内容改为：

```
auto lo
iface lo inet loopback

iface eth0 inet static
address 192.168.1.1
netmask 255.255.255.0
gateway 192.168.1.1
```

## 无线网络

有线网络需要受到网线的限制，现在无线网络已经非常的普及，通过一个无线网卡与无线路由器链接起来岂不是更加无拘无束，下面介绍一下无线网络的配置。

### 无线网卡驱动的确认

树莓派内置了很多无线网卡的驱动，大家可以在这个网站查找所支持的型号 [http://elinux.org/RPi_VerifiedPeripherals#USB_Wi-Fi_Adapters](http://elinux.org/RPi_VerifiedPeripherals#USB_Wi-Fi_Adapters)
笔者手里面有一个：8188CUS(网卡芯片)的验证支持。

验证方法：

将 USB 无线网卡插入树莓派 USB 接口（旧版系统会自动重启，新版不会），敲入：

```
$lsusb：
lsusb
```

提示

```
Bus 001 Device 002: ID 0424:9514 Standard Microsystems Corp.
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
Bus 001 Device 003: ID 0424:ec00 Standard Microsystems Corp.
Bus 001 Device 004: ID 0bda:8176 Realtek Semiconductor Corp. RTL8188CUS 802.11n WLAN Adapter
```


或者敲入：

```
$ifconfig:
```

提示

```
wlan0  Link encap:Ethernet  HWaddr 00:0b:81:87:e5:f9  
          UP BROADCAST MULTICAST  MTU:1500  Metric:1
          RX packets:0 errors:0 dropped:3 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000 
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)
```

说明树莓派支持你的无线网卡，可进行下一步设置。

### 无线网卡的配置

使用 vi 打开以下文件进行修改：

```
auto lo
iface lo inet loopback
iface eth0 inet dhcp
auto wlan0
allow-hotplug wlan0
iface wlan0 inet dhcp
wpa-ssid "jikeWiFi"
wpa-psk "hellworld"
#wpa-roam /etc/wpa_supplicant/wpa_supplicant.conf
```

重启即可连入:网络名为 jikeWiFi，密码为 helloworld 的网络，
如果想要设置为静态 ip，类似有线网卡，修改相应代码即可。

## 更新源测试

有线或者无线网络连通过，我们后面对软件更新是，需要首先进行更新列表更新，执行以下命令即可：

```
sudo apt-get update
```

显示示例：

```
Get:1 http://raspberrypi.collabora.com wheezy Release.gpg [836 B]              
Get:2 http://archive.raspberrypi.org wheezy Release.gpg [490 B]                
Get:3 http://raspberrypi.collabora.com wheezy Release [7,532 B]                
Get:4 http://mirrordirector.raspbian.org wheezy Release.gpg [490 B]            
Get:5 http://mirrordirector.raspbian.org wheezy Release [14.4 kB]              
Get:6 http://raspberrypi.collabora.com wheezy/rpi armhf Packages [2,214 B]     
Get:7 http://archive.raspberrypi.org wheezy Release [7,263 B]   
....
Ign http://mirrordirector.raspbian.org wheezy/rpi Translation-en_GB
Ign http://mirrordirector.raspbian.org wheezy/rpi Translation-en
Fetched 6,992 kB in 1min 12s (96.6 kB/s)      
Reading package lists... Done
```

改变更新源

```
sudo vi /etc/apt/sources.list
```
将默认的用`#`号屏蔽，改为

```
deb http://mirrors.ustc.edu.cn/raspbian/raspbian/   wheezy main contrib non-free rpi
```
或者

```
deb http://mirror.nus.edu.sg/raspbian/raspbian wheezy main contrib non-free rpi
```

或者用以下地址代替上面的地址栏

中山大学
[Raspbian http://mirror.sysu.edu.cn/raspbian/raspbian/](http://mirror.sysu.edu.cn/raspbian/raspbian/)

中国科学技术大学
[Raspbian http://mirrors.ustc.edu.cn/raspbian/raspbian/](http://mirrors.ustc.edu.cn/raspbian/raspbian/)

清华大学
[Raspbian http://mirrors.tuna.tsinghua.edu.cn/raspbian/raspbian/](http://mirrors.tuna.tsinghua.edu.cn/raspbian/)

华中科技大学
[Raspbian http://mirrors.hustunique.com/raspbian/raspbian/](http://mirrors.hustunique.com/raspbian/raspbian/)

[Arch Linux ARM http://mirrors.hustunique.com/archlinuxarm/](http://mirrors.hustunique.com/archlinuxarm/)

大连东软信息学院源（北方用户）

[Raspbian http://mirrors.neusoft.edu.cn/raspbian/raspbian/](http://mirrors.neusoft.edu.cn/raspbian/raspbian/)






