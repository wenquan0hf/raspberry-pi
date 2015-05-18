# 树莓派做 wifi 热点

原理：Pi 使用有线连入网络，然后接 USB 无线网卡作为热点，提供 Wifi 接入。

## USB 无线网卡驱动

如果接上 USB 无线网卡，使用 ifconfig 命令，能直接看到 wlan0，那么恭喜你，可以直接跳过这一步。
如果没有请查询一下树莓派支持的 USB 无线网卡型号，可参考以下网址：
[http://elinux.org/RPi_VerifiedPeripherals#USB_Wi-Fi_Adapters](http://elinux.org/RPi_VerifiedPeripherals#USB_Wi-Fi_Adapters)

## 修改wlan0为静态IP

相当于设置路由器 wlan 口 IP,即我们访问路由器通常使用的:`192.168.1.1`

```
sudo vim /etc/network/interfaces
```

把原来关于 wlan0 的注释掉：（可能跟这个不一样，跟 wlan0 有关的注释掉即可）。

```
#auto wlan0
#iface wlan0 inet dhcp
#wpa-ssid "360WiFi-li"
#wpa-psk "xiaolizi"
```

添加下面的

```
iface wlan0 inet static
address 192.168.0.1
netmask 255.255.255.0
gateway 192.168.0.1
```

完成之后需要重启

## 安装 hostapd

官方的 hostapd 不支持 8188CUS，后面需要重新卸载安装新的，
测试时这里必须先装旧的，然后后面卸了装新的，否则也不能用。

```
sudo apt-get install hostapd
```

### 编辑 hostapd 默认配置文件

```
sudo vim /etc/default/hostapd
```
找到 #DAEMON_CONF= ""，修改为：

```
DAEMON_CONF="/etc/hostapd/hostapd.conf"
```

### 然后编辑：`sudo vim /etc/hostapd/hostapd.conf`

增加以下代码：

```
[ruby] view plaincopy
# Basic configuration  
  
interface=wlan0  
ssid=raspberrywifi  
channel=1  
#bridge=br0  
  
# WPA and WPA2 configuration  
  
macaddr_acl=0  
auth_algs=1  
ignore_broadcast_ssid=0  
wpa=3  
wpa_passphrase=12345678  
wpa_key_mgmt=WPA-PSK  
wpa_pairwise=TKIP  
rsn_pairwise=CCMP  
  
# Hardware configuration  
  
driver=rtl871xdrv  
ieee80211n=1  
hw_mode=g  
device_name=RTL8192CU  
manufacturer=Realtek 
```
 
修改WiFi 名和密码

```
ssid=raspberrywifi
wpa_passphrase=12345678
```

### 保存退出，然后重启服务

```
sudo service hostapd restart
```

或者执行以下命令生效

```
sudo hostapd -dd /etc/hostapd/hostapd.conf
```

### 如果你使用的网卡提示一下信息

```
Configuration file: /etc/hostapd/hostapd.conf
nl80211: 'nl80211' generic netlink not found
Failed to initialize driver 'nl80211'
rmdir[ctrl_interface]: No such file or directory
```

那么，还是要使用第三方的 hostapd。

## 安装新的 hostapd

### 删除原来的 hostapd

```
sudo apt-get autoremove hostapd
```
### 下载第三方驱动并安装

```
wget https://github.com/jenssegers/RTL8188-hostapd/archive/v1.1.tar.gz
tar -zxvf v1.1.tar.gz
```

### 编译

```
cd RTL8188-hostapd-1.1/hostapd
sudo make
sudo make install
```

### 然后再重启服务，应该提示成功

```
$ sudo service hostapd restart
[ ok ] Stopping advanced IEEE 802.11 management: hostapd.
[ ok ] Starting advanced IEEE 802.11 management: hostapd.
```

### 将 hostapd 加入开机自启动

```
sudo service hostapd start
sudo update-rc.d hostapd enable
```
如果这里提示的是失败，重启后网络建立成功，用手机可以搜到这个网络

## 安装 DHCP 服务

以上步骤建立起了 WiFi 热点，但是无法自动获取 ip，需要以下步骤

```
sudo apt-get install udhcpd
```

### 编辑配置文件

```
sudo vim /etc/udhcpd.conf //修改以下信息，start和end是重点，注意跟第一步的静态ip在一个网段
start 192.168.0.20
end 192.168.0.200
interface wlan0
```

### 接下来编辑`/etc/default/udhcpd`并且将下面这行注释掉，以使DHCP Server正常工作

```
#DHCPD_ENABLED="no"
```

### 启动 dhcp 服务器

```
sudo service udhcpd start
sudo update-rc.d udhcpd enable
```
经过此步手机已经可以接入 WiFi 网络，并且自动获取 ip。

## 配置路由转发

理论上是经过这一步，手机可以通过共享树莓派的无线网络上网了，但是可能还存在一些路由转发问题。

### 设置路由映射规则

```
sudo iptables -F
sudo iptables -X
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
sudo iptables -A FORWARD -i eth0 -o wlan0 -m state --state RELATED,ESTABLISHED -j ACCEPT
sudo iptables -A FORWARD -i wlan0 -o eth0 -j ACCEPT
sudo bash -c iptables-save > /etc/iptables.up.rules
```

### 编辑`sudo vim /etc/network/if-pre-up.d/iptables`

添加下面两行代码：

```
#!/bin/bash
/sbin/iptables-restore < /etc/iptables.up.rules
```

保存退出，然后修改 iptables 权限：

```
sudo chmod 755 /etc/network/if-pre-up.d/iptables
```

### 开起内核转发

```
sudo vim /etc/sysctl.conf
```

找到下面两行：

```
#Uncomment the next line to enable packet forwarding for IPv4
#net.ipv4.ip_forward=1
```
把`net.ipv4.ip_forward`前面的`#`去掉，保存退出。

然后:

```
sudo sysctl -p
```

## 其它问题

最近经常发现无线网卡配置的 DHCP 不能发挥作用，经过排查发现给无线网卡指定的静态 IP 失败了，也就是说无线网卡没有 IP 导致 DHCP 无法工作，将`/etc/default/ifplugd`的内容修改配置如下：

```
INTERFACES="eth0"
HOTPLUG_INTERFACES="eth0"
ARGS="-q -f -u0 -d10 -w -I"
SUSPEND_ACTION="stop"
```