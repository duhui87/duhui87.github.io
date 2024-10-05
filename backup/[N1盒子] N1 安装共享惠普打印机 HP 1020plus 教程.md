```markdown
# 一、系统安装

N1可以用的系统非常多，目前经试验经典的 `Armbian_5.77_Aml-s905_Ubuntu_bionic_default_5.0.2_20190401.img` 兼容性不错。如果是用基于Debian的系统，权限方面容易出问题。

1. 用 `balenaEtcher` 软件把 `img` 文件刷到U盘，然后把 `uEnv` 文件的第一行改为：
   ```bash
   dtb_name=/dtb/meson-gxl-s905d-phicomm-n1.dtb
   ```
2. U盘插入，断电重启N1。如果N1已经是Linux系统，可以输入 `reboot update` 启动到U盘。
3. U盘系统启动完成后，输入 `./install.sh` 安装到 `emmc`。有一点比较奇怪，我在刷系统的时候，第一次找不到无线网卡，第二次USB口识别不了设备，第三次重刷就正常了。
4. 配置无线网络：
   ```bash
   armbian-config
   ```

---

# 二、安装驱动和服务

1. 修改国内源：
   ```bash
   sudo vi /etc/apt/sources.list
   ```
   将该文件改为：
   ```plaintext
   deb https://mirrors.ustc.edu.cn/ubuntu-ports/ bionic main restricted universe multiverse
   # deb-src https://mirrors.ustc.edu.cn/ubuntu-ports/ bionic main restricted universe multiverse
   deb https://mirrors.ustc.edu.cn/ubuntu-ports/ bionic-updates main restricted universe multiverse
   # deb-src https://mirrors.ustc.edu.cn/ubuntu-ports/ bionic-updates main restricted universe multiverse
   deb https://mirrors.ustc.edu.cn/ubuntu-ports/ bionic-backports main restricted universe multiverse
   # deb-src https://mirrors.ustc.edu.cn/ubuntu-ports/ bionic-backports main restricted universe multiverse
   deb https://mirrors.ustc.edu.cn/ubuntu-ports/ bionic-security main restricted universe multiverse
   # deb-src https://mirrors.ustc.edu.cn/ubuntu-ports/ bionic-security main restricted universe multiverse
   ```

2. 更新系统：
   ```bash
   sudo apt-get update
   ```

3. 安装CUPS打印服务：
   ```bash
   sudo apt-get install cups
   sudo vi /etc/cups/cupsd.conf
   ```
   将 `cupsd` 文件中 `localhost` 改成 `0.0.0.0`（0.0.0.0 意即任意IP地址），`Browsing off` 改成 `Browsing On`，并在四个地方分别添加 `Allow all`，具体如下：
   ```plaintext
   Listen 0.0.0.0:631
   Listen /var/run/cups/cups.sock

   # Show shared printers on the local network.
   Browsing On
   BrowseLocalProtocols dnssd

   # Restrict access to the server...
   <Location />
     Order allow,deny
     Allow all
   </Location>

   # Restrict access to the admin pages...
   <Location /admin>
     Order allow,deny
     Allow all
   </Location>

   # Restrict access to configuration files...
   <Location /admin/conf>
     AuthType Default
     Require user @SYSTEM
     Order allow,deny
     Allow all
   </Location>

   # Restrict access to log files...
   <Location /admin/log>
     AuthType Default
     Require user @SYSTEM
     Order allow,deny
     Allow all
   </Location>
   ```
   编辑完成，重启服务：
   ```bash
   service cups restart
   ```

4. 安装HP打印机驱动：
   网上教程普遍安装的是 `foo2zjs`，本文安装的是HP官方的 `HPLIP`：
   ```bash
   sudo apt-get install hplip
   ```
   然后还需要装插件，为了提升验证速度，修改 `validation.py` 文件：
   ```bash
   sudo vi /usr/share/hplip/base/validation.py
   ```
   ```python
   pgp_site = 'keyserver.ubuntu.com'
   ```
   安装插件，但实际上安装完 `hp1020` 打印机还是会缺少属性。不知道原因。
   ```bash
   hp-plugin -i
   ```
   输入该安全命令后，会出现一个文本框，里面有HPLIP的版本号，如 `PLUG-IN INSTALLATION FOR HPLIP 3.17.10`。然后如果网速够快，就选 `d` 直接网络安装，如果网速不行，就选 `p` 就地安装，就地安装所需的文件去下面这个网站：
   [Index of /download/printdriver/auxfiles/HP/plugins](https://openprinting.org/download/printdriver/auxfiles/HP/plugins)
   找到对应的版本，下载 `.run` 和 `.run.asc` 两个文件。

5. 安装AirPrint及打印机发现：
   ```bash
   apt-get install cups-browsed
   sudo service cups-browsed restart

   sudo apt-get -y install avahi-daemon avahi-discover libnss-mdns
   sudo service avahi-daemon restart
   ```

---

# 三、配置打印机

打开浏览器，网址输入 `N1的IP:631`，进入CUPS配置页面。安装打印机，驱动选 `hpcups`。选其 `hpijs` 其实也可以。Share框必须勾选上。

有两个指令很有用，一个是USB连接好打印机，输入 `lsusb` 命令可以查看是否已连接到USB：
```bash
root@aml:~# lsusb
Bus 002 Device 001: ID 1d6b:0003 Linux Foundation 3.0 root hub
Bus 001 Device 002: ID 03f0:2b17 Hewlett-Packard LaserJet 1020
Bus 001 Device 001: ID 1d6b:0002 Linux Foundation 2.0 root hub
```

另一个是 `hp-info -i`，可以查看是否正确连接了打印机，特别是1020是GDI打印机，打印机上电后需要下载固件到打印机，`deviceid` 可以显示固件是否已下载：
```plaintext
hp:/usb/HP_LaserJet_1020?serial=S44WTQ0

Device Parameters (dynamic data):
  Parameter                     Value(s)                                                  
  ----------------------------  ----------------------------------------------------------
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
                                1020;CMD:ACL;CLSRINTER;DES:HP LaserJet                  
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

---

# 四、使用

苹果、安卓或者电脑与N1连接入同一WiFi网，苹果和电脑可以默认搜索到打印机，打印无问题。安卓手机可以用系统打印、Mopria Print、Android CUPS Print三种打印连接方式，都可以打印。但是用WPS打印的时候始终都有字体放大，页面排版错误的问题。不管用哪种驱动和方式都无法解决，用Word打则可以。

另外，关闭打印机电源，再开，打印机等会红绿闪烁，表示打印机被识别，固件在下载了，这种状态下打印机会成功实现热插拔。然而，打印机如果是和N1同时通电，或者打印机通电状态下再开N1，则会出现打印机无法工作的现象，原因是HPLIP使用的 `libusb` 和 `usblp` 冲突，要么就不用HPLIP改用 `foo2zjs`，要么就屏蔽 `usblp`，方法如下：
```bash
vi /etc/modprobe.d/blacklistusblp.conf
```
编辑内容 `blacklist usblp`，设权限644。重启后即完全正常。

至此，所有的坑全部填完了，日常各种使用完全正常。
