# 树莓派 spi 液晶屏支持

树莓派官方支持 av 及 HDMI 输出，板子上预留了一个 csi 接口的液晶显示屏，但是一直没有相应的模组出现。在很多应用场合我们需要一些小型的液晶屏显示一些基本的信息，所以小屏驱动很是必要。

在 github 上有一个开源工程：notro/fbtft，完整的实现了 framebuffer 驱动，让树莓派完美支持 tft 液晶，下面对移植过程进行一个简单说明。

## 官网地址

工程首页：[https://github.com/notro](https://github.com/notro)

fbtft 源码：[https://github.com/notro/fbtft](https://github.com/notro/fbtft)

编译好的固件（基于3.12.25+）：[https://github.com/notro/rpi-firmware](https://github.com/notro/rpi-firmware)

使用说明(wiki)：[https://github.com/notro/fbtft/wiki](https://github.com/notro/fbtft/wiki)

## 使用编译好的固件(3.12.25+)

环境：树莓派

[https://github.com/notro/rpi-firmware](https://github.com/notro/rpi-firmware)

### 打开 SPI

树莓派默认 spi 是关掉的，我们需要打开

```
sudo vi /etc/modprobe.d/raspi-blacklist.conf
```
把下面这句话前面的`#`号删掉

```
blacklist spi-bcm2708
```

### 下载

- 以模块的形式编译进内核（需要手动或脚本加载模块）3.12.25+</br>
sudo REPO_URI=https://github.com/notro/rpi-firmware rpi-update
- 直接编译进内核</br>
sudo REPO_URI=https://github.com/notro/rpi-firmware BRANCH=builtin rpi-update
- 以模块的形式编译进内核</br>
sudo REPO_URI=https://github.com/notro/rpi-firmware BRANCH=latest rpi-update
- 直接下载压缩包，手动安装</br>
http://tronnes.org/downloads/2014-06-20-wheezy-raspbian-2014-07-25-fbtft-master-firmware.zip

### 配置

#### 手动加载模块

```
sudo modprobe fbtft_device name=adafruit22
```

name 后面的名字，要跟相应的液晶驱动芯片移植，使用的液晶芯片为：fb_ra8875，所以这里写的是：`er_tftm050_2`。其它芯片请查阅：https://github.com/notro/fbtft/blob/master/fbtft_device.c 文件。正常会提示以下信息。


```
fbtft_device:  SPI devices registered:
   fbtft_device:      spidev spi0.0 500kHz 8 bits mode=0x00
   fbtft_device:      spidev spi0.1 500kHz 8 bits mode=0x00
   fbtft_device:  'fb' Platform devices registered:
   fbtft_device:      bcm2708_fb id=-1 pdata? no
   fbtft_device: Deleting spi0.0
   fbtft_device:  GPIOS used by 'adafruit22':
   fbtft_device:    'reset' = GPIO25
   fbtft_device:    'led' = GPIO23
   fbtft_device:  SPI devices registered:
   fbtft_device:      spidev spi0.1 500kHz 8 bits mode=0x00
   fbtft_device:      fb_hx8340bn spi0.0 32000kHz 8 bits mode=0x00
   graphics fb1: fb_hx8340bn frame buffer, 176x220, 75 KiB video memory, 16 KiB buffer memory, fps=20, spi0.0 at 32 MHz
```

在/dev/目录下出现: /dev/fb1 设备

#### 自动加载模块

```
sudo vi  /etc/modules
```
加入以下语句，既可以在启动时自动加载模块。

```
spi-bcm2708
fbtft_device name=er_tftm050_2 speed=28000000 fps=25 verbose=0
```

### 使用

#### 手动启动 x11 和控制台到新的液晶屏

X Windows 显示在 fb1 上：

```
$FRAMEBUFFER=/dev/fb1 startx
```

Console 显示在 fb1 上：

```  
$con2fbmap 1 1
```

#### 自动登陆 x11

```
sudo vi /etc/inittab
    #1:2345:respawn:/sbin/getty --noclear 38400 tty1
    1:2345:respawn:/bin/login -f pi tty1 </dev/tty1 >/dev/tty1 2>&1

sudo vi /etc/rc.local
    su -l pi -c "env FRAMEBUFFER=/dev/fb1 startx &"
```

### 测试

#### 将 fb0 上的内容直接拷贝到 fb1 上,fb0 和 fb1 同步

[https://github.com/notro/fbtft/wiki/Framebuffer-use#framebuffer-mirroring](https://github.com/notro/fbtft/wiki/Framebuffer-use#framebuffer-mirroring)

```
$git clone https://github.com/tasanakorn/rpi-fbcp
$cd rpi-fbcp/
$mkdir build
$cd build/
$cmake ..
$make
$sudo install fbcp /usr/local/bin/fbcp
```

启动：fbcp &</br>
关闭fbcp：killall fbcp

#### 启动时使用 fb1

```
$sudo apt-get install xserver-xorg-video-fbdev

$sudo vi /usr/share/X11/xorg.conf.d/99-fbdev.conf
```

加入以下语句：

```
Section "Device"  
  Identifier "myfb"
  Driver "fbdev"
  Option "fbdev" "/dev/fb1"
EndSection
```

启动：startx

测试：

```
apt-get -y install fbi
fbi -d /dev/fb1 -T 1 -noverbose -a test.jpg
```

## 由内核及源码编译

#### 下载、编译内核源码

请见[《树莓派内核编译与固件升级》](software.md)

#### 下载、编译 fbtft 源码

```
$cd linux(进入下载好的内核源码目录)
$cd drivers/video
$git clone https://github.com/notro/fbtft.git
```

修改内核源码的 Kconfig 及 Makefine

```
 Add to drivers/video/Kconfig:   source "drivers/video/fbtft/Kconfig"
 Add to drivers/video/Makefile:  obj-y += fbtft/
$make menuconfig(在配置界面加入所选用液晶的驱动支持)
```

```
Device Drivers  --->   
 Graphics support  --->   
<M> Support for small TFT LCD display modules  --->  
  
<M>   FB driver for the HX8353D LCD Controller  
<M>   FB driver for the ILI9320 LCD Controller                                                 
<M>   FB driver for the ILI9325 LCD Controller                                   
<M>   FB driver for the ILI9340 LCD Controller                                     
<M>   FB driver for the ILI9341 LCD Controller                              
< >     FB driver for the ILI9481 LCD Controller                                             
<M>   FB driver for the ILI9486 LCD Controller                                         
<M>   FB driver for the PCD8544 LCD Controller                                       
<M>   FB driver for the RA8875 LCD Controller  
```



