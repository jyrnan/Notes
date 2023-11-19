# How to Install Linux on a Mac : HelloTech How

[https://www.hellotech.com/guide/for/how-to-install-linux-on-mac](https://www.hellotech.com/guide/for/how-to-install-linux-on-mac)

Linux is an open-source operating system that you can install on your computer for free. It offers several advantages over Windows and Mac, such as flexibility, privacy, better security, and easy customization. If you have a Mac, and you would like to explore a new, open-source OS, check out our guide on how to install Linux on a Mac.

If you want to dual boot Linux and macOS, you will need to create a space for your Linux OS to live. In order to do this, you will have to partition your main hard drive. Here’s how to do that:

## How to Partition your Mac Hard Drive

1. **Open your Applications folder and click Utilities.**
2. **Then open Disk Utility**.
3. **Next, click View in the top left corner of the window**.
4. **Then select Show All Devices.**
5. **Select your macOS partition and then click Partition**. You can find this button at the top of the window. If you are using a newer Mac, you might be asked to add a volume. You can click Partition on the pop-up.
    
    ![How%20to%20Install%20Linux%20on%20a%20Mac%20HelloTech%20How%20b844d531ca7d42f288c78962a73b9998/mac-disk-utility-format-drive-partition.jpg](How%20to%20Install%20Linux%20on%20a%20Mac%20HelloTech%20How%20b844d531ca7d42f288c78962a73b9998/mac-disk-utility-format-drive-partition.jpg)
    
    mac disk utility format drive partition
    
6. **Set the partition size you need for your Linux OS**. You can do this by dragging the little white circles on the edge of the larger circle, or you can enter the size in the box next to Size. The gray side of the pie will be your Mac space, while the blue space will be your new partition. It is recommended that you set aside at least 20GB for Linux. You can also add other information here as well, such as the partition name. Take note of the size of your partition, you will need this information later.
7. **Format the new partition as MS-DOS (FAT).**
8. **Click Apply**. This will cause your computer to become unresponsive as the partition is created.
    
    ![How%20to%20Install%20Linux%20on%20a%20Mac%20HelloTech%20How%20b844d531ca7d42f288c78962a73b9998/ms-dos-fat-mac-disk-utility-partition.jpg](How%20to%20Install%20Linux%20on%20a%20Mac%20HelloTech%20How%20b844d531ca7d42f288c78962a73b9998/ms-dos-fat-mac-disk-utility-partition.jpg)
    
    ms dos fat mac disk utility partition
    
9. **Click Partition and Continue when further prompted and wait for the partition process to finish.**

## How to Make a Bootable USB on a Mac

1. **Download a Linux distro.** An ISO file is a disk image. Some of the top options are [Ubuntu](https://ubuntu.com/download/desktop), [Mint](https://linuxmint.com/download.php), or [Deepin](https://www.deepin.org/en/). They are free to download from each distribution’s main website. In this article, we are using Ubuntu.
2. **Open Disk Utility, select your USB drive, and click *Erase*.**
3. **Then format your USB drive as MS-DOS (FAT) and the scheme as GUID Partition Map**. You can do this in Disk Utility by selecting your drive and clicking Erase.
    
    ![How%20to%20Install%20Linux%20on%20a%20Mac%20HelloTech%20How%20b844d531ca7d42f288c78962a73b9998/erase-usb-bootable-linux-mac.jpg](How%20to%20Install%20Linux%20on%20a%20Mac%20HelloTech%20How%20b844d531ca7d42f288c78962a73b9998/erase-usb-bootable-linux-mac.jpg)
    
    erase usb bootable linux mac
    
4. **Download the Etcher app**. This will allow you to burn your Linux distro onto a USB flash drive. You can [download](https://www.balena.io/etcher/) Etcher for free here.
5. **Move the Etcher app to your Applications folder.** If you get a warning saying that you are not able to open the application, you will have to go to System Preferences > Security and Privacy > General. Then click the lock, enter your password, and click Open Anyway.
    
    ![How%20to%20Install%20Linux%20on%20a%20Mac%20HelloTech%20How%20b844d531ca7d42f288c78962a73b9998/open-etcher-anyway-mac.jpg](How%20to%20Install%20Linux%20on%20a%20Mac%20HelloTech%20How%20b844d531ca7d42f288c78962a73b9998/open-etcher-anyway-mac.jpg)
    
    open etcher anyway mac
    
6. **Open Etcher, click Select Image, and select your Linux ISO**. This is the file you downloaded in step one.
7. **Click Select target and choose your USB drive**.
8. **Then click Flash**. If you get an error message, and you’re running a newer OS, go to Applications > Utilities and open the Terminal app. Then enter the following command and hit enter:

```
sudo /Applications/balenaEtcher.app/Contents/MacOS/balenaEtcher
```

If you still get an error message that says the disk you inserted was not readable, click ignore.

## How to Install Linux on a Mac

Once you’ve made a partition, you can then continue with the rest of the Linux installation setup below. But before you begin, you will need a USB mouse and keyboard. This is because Linux will not detect your drivers if you have a newer Mac.

1. **Switch off your Mac computer.**
2. **Plug the bootable Linux USB drive into your Mac.**
3. **Turn on your Mac while holding down the Option key.** You will then see the boot manager with a list of available devices you can boot from.
4. **Select your USB stick and hit enter.** This will be named EFI boot or EFI drive.
    
    ![How%20to%20Install%20Linux%20on%20a%20Mac%20HelloTech%20How%20b844d531ca7d42f288c78962a73b9998/Select-your-USB-stick-and-hit-enter.jpg](How%20to%20Install%20Linux%20on%20a%20Mac%20HelloTech%20How%20b844d531ca7d42f288c78962a73b9998/Select-your-USB-stick-and-hit-enter.jpg)
    
    Select your USB stick and hit enter
    
5. **Then select Install from the GRUB menu.** You can also select *Try Without Installing*.
6. **Follow the on-screen installation instructions**. Select Normal Installation.
7. **On the *Installation Type* window, choose *Something else*.** This is an option that will let you choose the partition you made earlier for installing your new Linux OS. If you choose the other options, you risk installing Linux over your current operating system, which would also erase all your files and folders.
    
    ![How%20to%20Install%20Linux%20on%20a%20Mac%20HelloTech%20How%20b844d531ca7d42f288c78962a73b9998/install-linux-mac-something-else.jpg](How%20to%20Install%20Linux%20on%20a%20Mac%20HelloTech%20How%20b844d531ca7d42f288c78962a73b9998/install-linux-mac-something-else.jpg)
    
    install linux mac something else
    
8. **Choose the partition you made**. This is the partition that is the same size as the one that you made earlier.
9. **Then click Change.** You will see this under the list of devices.
10. **In the *Use as* drop-down, select ext4 journaling file system.**
11. **In the *Mount Point* drop-down, select forward slash and click OK.** 
    
    ![How%20to%20Install%20Linux%20on%20a%20Mac%20HelloTech%20How%20b844d531ca7d42f288c78962a73b9998/change-linux-partition-mac-install.jpg](How%20to%20Install%20Linux%20on%20a%20Mac%20HelloTech%20How%20b844d531ca7d42f288c78962a73b9998/change-linux-partition-mac-install.jpg)
    
    change linux partition mac install
    
12. **Click *Continue* when prompted.**
13. **Then click *Install Now*.**
14. **Next, click *Continue* on succeeding warning pop-ups.**
15. **Enter your time zone, choose your keyboard layout, and click Continue.**
16. **Set up your Linux account.** Type in your name, computer name, username, and password.
17. **Click *Continue*.**
18. **Wait for the installation process to start and finish.**
19. **Restart your Mac and hold down the Option key while it does.**
20. **Choose your Linux OS to start using it.**

After installing the OS and exploring it, make sure that your applications are working properly.

Now that you know how to install Linux on your Mac, check out our guide on [how to install Linux on a Windows 10](https://hellotechguide.wpengine.com/for/how-to-install-linux-on-mac) computer.

### Unlock the rest of the article for free by entering your email address below.