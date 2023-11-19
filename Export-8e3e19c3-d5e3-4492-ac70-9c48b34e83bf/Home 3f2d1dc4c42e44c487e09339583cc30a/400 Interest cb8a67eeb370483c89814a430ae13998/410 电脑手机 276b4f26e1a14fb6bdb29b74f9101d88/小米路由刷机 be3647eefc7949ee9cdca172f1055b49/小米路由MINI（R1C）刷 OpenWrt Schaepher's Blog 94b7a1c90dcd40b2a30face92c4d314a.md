# 小米路由MINI（R1C）刷 OpenWrt | Schaepher's Blog

本文包括三个部分：

1. 连接路由器 Shell
2. 下载和写入 OpenWrt 升级包
3. 配置 OpenWrt

## 连接路由器 Shell

由于现在小米禁止刷官方的 SSH 开启工具，因此需要使用另外的方法进入 Shell。

> 
> 
> 
> 官方的 SSH 开启工具：[https://d.miwifi.com/rom/ssh](https://d.miwifi.com/rom/ssh)
> 

这里使用 openwrt 提供的方法。

> 
> 
> 
> openwrt xiaomi mini：[https://openwrt.org/toh/xiaomi/mini](https://openwrt.org/toh/xiaomi/mini)
> 
1. 路由器固件降级 
    1. 下载官方旧版本固件（新版的固件可能没法用以下方法）： [http://bigota.miwifi.com/xiaoqiang/rom/r1cm/miwifi_r1cm_firmware_b9d56_2.7.11.bin](http://bigota.miwifi.com/xiaoqiang/rom/r1cm/miwifi_r1cm_firmware_b9d56_2.7.11.bin)
    2. 连接路由器 WIFI 或者网线直连
    3. 打开浏览器进入 192.168.31.1 ，这个是路由的管理界面
    4. 选择【常用设置】-> 【系统信息】 -> 【升级】，选择刚才下载的文件，确定。 此时会提示系统降级最好删除配置文件，勾选并继续。
    5. 等执行完，路由器会重启，并闪烁黄灯。一直等到蓝灯常亮，表示降级完毕。
2. 开启 telnet （现在还开不了 SSH） 
    1. 打开浏览器进入 192.168.31.1 ，配置并等待初始化完成。
    2. 此时 url 会包含 stok=xxxxx，把 xxxx 复制出来，以下用 表示复制出来的部分。注：一定要等上面初始化完成，否则下面的命令无法执行。
    3. 复制下面这串： `http://192.168.31.1/cgi-bin/luci/;stok=<STOK>/api/xqnetwork/set_wifi_ap?ssid=whatever&encryption=NONE&enctype=NONE&channel=1%3B%2Fusr%2Fsbin%2Ftelnetd` 贴到浏览器（注意替换 ），回车。这个用来开启 telnet。 执行结束会提示：`{“msg”:“未能連線到指定Wi-Fi(Probe timeout)”,“code”:1616}` 虽然信息是错误，但实际上是成功。
    4. 复制下面这串： `http://192.168.31.1/cgi-bin/luci/;stok=<STOK>/api/xqsystem/set_name_password?oldPwd=<CURRENTPASS>&newPwd=<NEWPASS>` 贴到浏览器（注意替换 （当前路由登陆密码） （新的登陆密码）），回车。 这个用来重新设置密码。 执行结束会提示：`{“code”:0}`。 此时 telnet 已开启。
3. 打开 Windows 的 cmd，并连接路由器： `telnet 192.168.31.1 23` 如果提示找不到 telnet，需要在控制面板的【程序和功能】->【启用或关闭 Windows 功能】里面找到 【Telnet 客户端】，前面的打勾，并点【确定】。 连接时的用户名为 root，密码为刚才的

## 备份 MTD

1. 插入 U 盘，在 telnet 里面进入 U 盘文件夹。在 /extxxxx/ext4 里面。这里的 xxxx 根据不同情况可能不同，你可以 `cd / && ls` 看到以 ext 开头的文件夹
2. 执行以下命令： 
    
    
    ```
    1
    
    ```
    
    ```
    for name in $(grep -v 'dev' /proc/mtd | awk -F ':' '{print $1}'); do dd if=/dev/$name of=/extxxxx/ext4/$name.bin; done
    
    ```
    

## 刷引导 BootLoader（不死 Breed）

主要是为了避免把路由器刷坏，没法恢复。只要刷成功，以后就不用怕了。而且刷固件用界面操作也比较方便。

1. 进入官网，找到 breed-mt7620-xiaomi-mini.bin官方网站：[https://breed.hackpascal.net/](https://breed.hackpascal.net/)将文件下载到 U 盘
2. 进入 U 盘，执行命令写入：`mtd -r write /extxxxx/ext4/breed-mt7620-xiaomi-mini.bin Bootloader`
3. 等待路由器重启
4. 重启后会亮红灯闪烁，等待红灯常亮，即表示成功。

进入 Bread 的方式：

1. 关掉路由器电源
2. 按住 reset
3. 接通路由器电源，等待 3 秒，灯闪烁，再放开 reset
4. 打开 192.168.1.1

## 刷 OpenWrt

### 法一：Bread 刷 OpenWrt

1. 获取 OpenWrt 升级包下载地址 
    1. 回到 OpenWrt 的页面 [https://openwrt.org/toh/xiaomi/mini](https://openwrt.org/toh/xiaomi/mini)
    2. 找到【OpenWrt support】这一块，复制【Firmware OpenWrt Upgrade】下面的链接
    3. 下载这个以 `ramips-mt7620-miwifi-mini-squashfs-sysupgrade.bin` 结尾的文件
2. 进入 Bread
3. 在【固件启动设置】里，将类型设置为【小米 Mini】
4. 在【固件备份】里，都点一遍
5. 在【固件更新】里，在【固件】一栏选择刚才下载的 bin。
6. 点上传，等待上传完毕，路由器会自动重启。
7. 重启后会亮红灯闪烁，等待红灯常亮，即表示成功。

### 法二：下载和写入 OpenWrt 升级包

1. 获取 OpenWrt 升级包下载地址  
    1. 回到 OpenWrt 的页面 [https://openwrt.org/toh/xiaomi/mini](https://openwrt.org/toh/xiaomi/mini)
    2. 找到【OpenWrt support】这一块，复制【Firmware OpenWrt Upgrade】下面的链接
2. 升级  
    1. 刚刚连接的 telnet 执行：
        
        
        ```
        1
        2
        
        ```
        
        ```
        cd /tmp
        wget http://downloads.openwrt.org/releases/18.06.4/targets/ramips/mt7620/openwrt-18.06.4-ramips-mt7620-miwifi-mini-squashfs-sysupgrade.bin
        
        ```
        
    2. 检查 MTD `cat /proc/mtd` 你可以看到有一行包含 OS1
    3. 写入升级包 `mtd -r write 刚刚下载的文件名 OS1`
    4. 写入完成后，路由器会自动重启
    5. 重启后会亮红灯闪烁，等待红灯常亮，即表示成功。

## 配置 OpenWrt

以下配置完成后，记得在【系统】->【备份/升级】->【动作】->【生成备份】创建备份。以后重新刷 OpenWrt 的时候，就可以直接导入。不用再做配置。

1. 进入 OpenWrt 界面启用 WIFI  
    1. 刚装完后，WIFI 没有默认开启。所以需要用网线连接路由器的 LAN 口。如果电脑连了其他路由器的 WIFI，则先断掉。
    2. 进入 192.168.1.1 。刚开始会要求设置密码。点【Login】按钮，再点【Go to password configuration…】进入设置界面。 在 Password 和 Confirmation 输入密码，点 【Save & Apply】。
    3. 进入顶部的 【Network】->【Wireless】，选择一个，点击 【Enable】就开启 WIFI 了。
    4. 点【Edit】进去设置 ESSID（WIFI连接名称）。 在【Wireless Security】一栏的 Encryption 选择 WPA2-PSK，然后在 Key 一栏填入密码。点【Save & Apply】。
2. 开启 SSH  
    1. 进入顶部的【System】->【Administration】
    2. 在 Dropbear Instance 下面的 Interface 选择 lan
    3. Port 设置一个，比如 55555
    4. 确保勾选了【Password authentication】和【Allow root logins with password】
    5. 点【Save & Apply】
3. 修改路由地址   
    
    我这里是路由器用线连接光猫，光猫的地址是 192.168.1.1，无法修改。OpenWrt 也是 192.168.1.1。如果输入 192.168.1.1 会进入 OpenWrt 管理界面，而不是光猫的。所以可以选择将 OpenWrt 的地址改掉。
    
    1. 第一种方法是在管理界面的【Network】->【Interface】->【LAN】，把【IPv4 address】这一栏改成类似 192.168.23.1。点【Save & Apply】 
    2. 第二种方法是进入 SSH，编辑文件：`vim /etc/config/network`    
        
        修改 `config interface 'lan'` 这一区块的 `option ipaddr`，修改后类似于：
        
        ```
        1
        2
        3
        4
        5
        6
        7
        
        ```
        
        ```
        config interface 'lan'
             option type 'bridge'
             option ifname 'eth0.1'
             option proto 'static'
             option ipaddr '192.168.23.1'
             option netmask '255.255.255.0'
             option ip6assign '60'
        
        ```
        
        然后执行 `/etc/init.d/network reload`，等一会儿就可以通过 `192.168.23.1` 访问路由器了。我试了好多次第一种方法，总是保存不了，用这个方法才成功。
        

## 安装管理界面中文包

网上说的安装方法大多过时，在 OpenWrt 18.06.4 版本，需要执行以下命令安装：

从下面这个文档找到的：[https://openwrt.org/packages/pkgdata/luci-i18n-base-lang](https://openwrt.org/packages/pkgdata/luci-i18n-base-lang)

## USB 支持

1. 安装  
    
    
    ```
    1
    2
    3
    4
    5
    6
    7
    8
    
    ```
    
    ```
    opkg install kmod-usb-core \
    			kmod-usb-uhci \
    			kmod-usb-storage \
    			kmod-usb2 \
    			kmod-usb-ohci \
    			block-mount \
    			mount-utils \
    			fdisk
    
    ```
    
    如果 U 盘或移动硬盘是 FAT32 的，就装 `kmod-fs-vfat`；如果是 NTFS 的，就装 `ntfs-3g`。
    
2. 挂载  
    1. 执行 `mkdir /mnt/usb` 创建等下的挂载目录
    2. 插入 U 盘
    3. 执行 `fdisk -l | grep "^/"` 会看到与 U 盘容量差不多的分区。这里假设为 `/dev/sda4`
    4. 执行 `mount /dev/sda4 /mnt/usb`

这样进入 `/mnt/usb` 就能看到 U 盘的文件了

## 主要参考文档

- 基础知识：[https://www.zoulei.net/2016/05/05/openwrt_recovery_you_need_to_know/](https://www.zoulei.net/2016/05/05/openwrt_recovery_you_need_to_know/)
- 修改路由地址：[https://www.cnblogs.com/double-win/p/3841017.html](https://www.cnblogs.com/double-win/p/3841017.html)
- Breed：[https://www.jianshu.com/p/cab3062ef920](https://www.jianshu.com/p/cab3062ef920)
- U 盘：[https://jingyan.baidu.com/article/5225f26b6b273fe6fa090829.html](https://jingyan.baidu.com/article/5225f26b6b273fe6fa090829.html)
- U 盘：[https://www.cnblogs.com/double-win/p/3841801.html](https://www.cnblogs.com/double-win/p/3841801.html)
- MTD：[http://blog.chinaunix.net/uid-28790518-id-5082378.html](http://blog.chinaunix.net/uid-28790518-id-5082378.html)