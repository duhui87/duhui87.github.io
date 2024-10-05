# 一、系统安装

N1可以用的系统非常多，目前经试验经典的Armbian_5.77_Aml-s905_Ubuntu_bionic_default_5.0.2_20190401.img兼容性不错。如果是用基于Debian的系统，权限方面容易出问题。

1、用balenaEtcher软件把img文件刷到U盘，然后把uEnv文件的第一行改为：

```
dtb_name=/dtb/meson-gxl-s905d-phicomm-n1.dtb
```

2、U盘插入，断电重启N1。如果N1已经是linux系统，可以输入reboot update启动到U盘。

3、U盘系统启动完成后，输入./install.sh安装到emmc。有一点比较奇怪，我在刷系统的的时候，第一次找不到无线网卡，第二次usb口识别不了设备，第三次重刷就正常了。

4、配置无线网络

```
armbian-config
```

---

# 二、安装驱动和服务

## 1、修改国内源

```
sudo vi /etc/apt/sources.list
```

将该文件改为

```deb [https://mirrors.ustc.edu.cn/ubuntu-ports/](https://mirrors.ustc.edu.cn/ubuntu-ports/) bionic main restricted universe multiverse
# deb-src [https://mirrors.ustc.edu.cn/ubuntu-ports/](https://mirrors.ustc.edu.cn/ubuntu-ports/) bionic main main restricted universe multiverse

deb [https://mirrors.ustc.edu.cn/ubuntu-ports/](https://mirrors.ustc.edu.cn/ubuntu-ports/) bionic-updates main restricted universe multiverse

# deb-src [https://mirrors.ustc.edu.cn/ubuntu-ports/](https://mirrors.ustc.edu.cn/ubuntu-ports/) bionic-updates main restricted universe multiverse

deb [https://mirrors.ustc.edu.cn/ubuntu-ports/](https://mirrors.ustc.edu.cn/ubuntu-ports/) bionic-backports main restricted universe multiverse

# deb-src [https://mirrors.ustc.edu.cn/ubuntu-ports/](https://mirrors.ustc.edu.cn/ubuntu-ports/) bionic-backports main restricted universe multiverse

deb [https://mirrors.ustc.edu.cn/ubuntu-ports/](https://mirrors.ustc.edu.cn/ubuntu-ports/) bionic-security main restricted universe multiverse

# deb-src [https://mirrors.ustc.edu.cn/ubuntu-ports/](https://mirrors.ustc.edu.cn/ubuntu-ports/) bionic-security main restricted universe multiverse
```

---

## 2、更新系统

```
sudo apt-get update
```

## 3、安装cups打印服务

```sudo apt-get install cups
sudo vi /etc/cups/cupsd.conf
```

将cupsd文件中localhost改成0.0.0.0（0.0.0.0意即任意IP地址），Browsing off改成Browsing On，并在四个地方分别添加Allow all，具体如下：

```Listen 0.0.0.0:631
Listen /var/run/cups/cups.sock

# Show shared printers on the local network.

Browsing On
BrowseLocalProtocols dnssd

# Restrict access to the server...



Order allow,deny

Allow all



# Restrict access to the admin pages...



Order allow,deny

Allow all



# Restrict access to configuration files...



AuthType Default

Require user @SYSTEM

Order allow,deny

Allow all



# Restrict access to log files...



AuthType Default

Require user @SYSTEM

Order allow,deny

Allow all


```


编辑完成，重启服务

```
service cups restart
```

---



## 4、安装HP打印机驱动

网上教程普遍安装的是foo2zjs，本文安装的是HP官方的HPLIP

```
sudo apt-get install hplip
```

然后还需要装插件，为了提升验证速度，修改validation.py文件

```sudo vi /usr/share/hplip/base/validation.py
pgp_site = 'keyserver.ubuntu.com'
```


安装插件，但实际上安装完hp1020打印机还是会缺少属性。不知道原因。

```
hp-plugin -i
```

输入该安全命令后，会出现一个文本框，里面有HPLIP的版本号。

如PLUG-IN INSTALLATION FOR HPLIP 3.17.10

然后如果网速够快，就选d直接网络安装，如果网速不行，就选p就地安装，就地安装所需的文件去下面这个网站：
[https://www.openprinting.org/download/printdriver/auxfiles/HP/plugins/?C=N;O=D](https://)

找到对应的版本，下载.run和run.asc两个文件

## 5、安装airpint及打印机发现。

```apt-get install cups-browsed
sudo service cups-browsed restart

sudo apt-get -y install avahi-daemon avahi-discover libnss-mdns

sudo service avahi-daemon restart
```

---



# 三、配置打印机

打开浏览器，网址输入N1的IP:631，进入cups配置页面。

安装打印机，驱动选hpcups。选其hpijs其实也可以。Share框必须勾选上。

有两个指令很有用，一个是USB连接好打印机，输入lsusb命令可以查看是否已连接到usb。

```
root@aml:~# lsusb
Bus 002 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub

Bus 001 Device 002: ID 03f0:2b17 Hewlett-Packard LaserJet 1020

Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
```

一个是hp-info -i，可以查看是否正确连接了打印机，特别是1020是GDI打印机，打印机上电后需要下载固件到打印机，deviceid可以显示固件是否已下载。

```
hp:/usb/HP_LaserJet_1020?serial=S44WTQ0
Device Parameters (dynamic data):
Parameter                     Value(s)

---

agent1-ack                    False
agent1-desc                   Black toner cartridge
agent1-dvc                    0
agent1-health                 0
agent1-health-desc            Good/OK
agent1-hp-ink                 False
agent1-id                     0
agent1-kind                   4
agent1-known                  False
agent1-level                  100
agent1-level-trigger          0
agent1-sku                    Q2612A
agent1-type                   1
agent1-virgin                 False
back-end                      hp
cups-printers                 ['HP_LaserJet_1020']
cups-uri                      hp:/usb/HP_LaserJet_1020?serial=S44WTQ0
dev-file
device-state                  1
device-uri                    hp:/usb/HP_LaserJet_1020?serial=S44WTQ0
deviceid                      MFG:Hewlett-Packard;MDL:HP LaserJet
1020;CMD:ACL;CLS![](https://www.right.com.cn/FORUM/static/image/smiley/default/tongue.gif)

RINTER;DES:HP LaserJet
1020;FWVER:20080222;
duplexer                      0
error-state                   0
host
in-tray1                      1
in-tray2                      1
is-hp                         True
media-path                    1
panel                         0
panel-line1
panel-line2
photo-tray                    0
port                          1
r                             0
revision                      254
rg                            000
rr                            000000
rs                            000000000
serial                        S44WTQ0
status-code                   1000
status-desc                   Idle
supply-door                   1
top-door                      1
```



# 四、使用

苹果、安卓或者电脑与N1连接入同一wifi网，苹果和电脑可以默认搜索到打印机，打印无问题。

安卓手机可以用系统打印、Mopria Print、Android CUPS Print三种打印连接方式，都可以打印。但是用wps打印的时候始终都有字体放大，页面排版错误的问题。不管用哪种驱动和方式都无法解决，用word打则可以。

另外，关闭打印机电源，再开，打印机等会红绿闪烁，表示打印机被识别，固件在下载了，这种状态下打印机会成功实现热插拔。

然而，打印机如果是和N1同时通电，或者打印机通电状态下再开N1，则会出现打印机无法工作的现象，原因是hplip使用的libusb和usblp冲突，要么就不用hplip改用foo2zjs，要么就屏蔽usblp，方法如下：

```
vi /etc/modprobe.d/blacklistusblp.conf，编辑内容blacklist usblp，设权限644。重启后即完全正常。
```

至此，所有的坑全部填完了，日常各种使用完全正常。
