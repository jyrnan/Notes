# Chapter 4 covers commands for working with files

# Understanding File Types

Files that provide access to the hardware components on your computer are referred to as *device files*. There are character and block devices. You can use *hard links* and *soft links* to make the same file accessible from different locations. Regular users will also use, though less directly, *named pipes* and *sockets*, which provide access points for processes to communicate with each other.

## Using Regular Files

`file` 可以列出文件类型

- 创建空白文件的方式

```bash
$ touch /tmp/newfile.txt                 //Create a blank file 
$ cat /dev/null > /tmp/newfile2.txt      //Create an empty file
```

## Using Directories

- 创建文件夹的命令及参数

```bash
$ mkdir /tmp/new              //Create “new” directory in /tmp
$ mkdir -p /tmp/a/b/c/new     //Create parent directories as needed for “new” 
$ mkdir -m 700 /tmp/new2      //Create new2 with drwx——— permissions
```

- 通过属性的第一字母可以判断是一个目录

```bash
$ ls -ld /tmp/new
drwxr-xr-x 2 chris chris 4096 Sep 5 14:53 /tmp/new
```

## Using symbolic and hard Links

区别是什么？

> A hard link is essentially a synced carbon copy of a file that refers directly to the inode of a file. Symbolic links on the other hand refer directly to the file which refers to the inode, a shortcut.
> 

---

> Links are of two types: soft links (symbolic links) or hard links.
> 
> 
> **Soft Links** (symbolic links)
> 
> You can make links to files and directories, and you can create links (shortcuts) on different partitions and with a different inode number than the original.
> 
> If the *real* copy is deleted, the link **will not work**.
> 
> **Hard Links**
> 
> Hard links are for files only; you cannot link to a file on a different partition with a different inode number.
> 
> If the *real* copy is deleted, the **link will work**, because it accesses the underlying data which the real copy was accessing.
> 

[理解inode - 阮一峰的网络日志](../../300%20Learning%2085ac233e054e457b9ea0166355f87cb8/311%20Fundamental%20e4155938509041c8af1f134529265f3a/%E7%90%86%E8%A7%A3inode%20-%20%E9%98%AE%E4%B8%80%E5%B3%B0%E7%9A%84%E7%BD%91%E7%BB%9C%E6%97%A5%E5%BF%97%20c187b62e8faa42e8bc64ce3eec93becc.md)

hardlink只能是文件，感觉是同一个文件的镜像？

symbo link是文件的连接而已

```bash
$ touch myfile
$ ln myfile myfile-hardlink
$ ln -s myfile myfile-symlink
$ ls -li myfile*
292007 -rw-rw-r-- 3 francois francois 0 Mar 25 00:07 myfile
292007 -rw-rw-r-- 3 francois francois 0 Mar 25 00:07 myfile-hardlink
292008 lrwxr-xr-x 2 francois francois 6 Mar 25 00:09 myfile-symlink -> myfile
```

## Using Device Files

device files are stored in the /dev directory.

Devices were generally divided into **block devices** (such as storage media) and **character devices** (such as serial ports and terminal devices). Mac OS X, however, uses only character devices to communicate with the hardware.

```bash
$ ls -l /dev/*
```

## Using named pipes and sockets

When you want to allow one process to send information to another process, you can simply pipe (|) the output from one to the input of the other.

- Named pipes are typically used for interprocess communication on the local system,
- whereas sockets can be used for processes to communicate over a network.

一般在/tmp下面建立各种pipe和sokets

```bash
$ ls -l /tmp/.TV-chris/tvtimefifo-local /tmp/.X11-unix/X0
prw------- 1 chris chris 0 Sep 26 2007 /tmp/.TV-chris/tvtimefifo-local 
srwx------ 1 chris wheel 0 Sep 4 01:30 /tmp/fam-chris/fam-
```

还可以创建自己的命名管道

```bash
$ mkfifo mypipe
$ ls -l mypipe
prw-r--r-- 1 chris chris 0 Sep 26 00:57 mypipe
```

<aside>
💡 所以：p开通就是pipe， s开头就是socket？ d开头就是directory！

</aside>

# Setting File and  Directory

### 权限类型表示

![Untitled](Chapter%204%20covers%20commands%20for%20working%20with%20files%2032371ac794b641b7b1aa2adf5562d48a/Untitled.png)

```bash
$ mkdir /tmp/test
$ echo “some text” > /tmp/test/hello.txt
$ ls -ld /tmp/test/ /tmp/test/hello.txt
drwxrwxr-x 2 francois wheel 4096 Mar 21 13:11 /tmp/test 
-rw-r--r-- 2 francois wheel 10 Mar 21 13:11 /tmp/test/hello.txt
```

`drwxrwxr-x`第一个字母的确代表文件的类型。

> first character of the long listing shows / tmp/test as a directory (d) and hello.txt as a file (-).
Other types of files that would appear as the first character include character devices (c), block devices (b) or symbolic links (l), named pipes (p), and sockets (s).
> 

后面9个字母代表文件或目录的权限

### 更改权限

```bash
chmod -R 700 /tmp/test Open permission only to owner below /tmp/test # chmod -R 000 /tmp/test Close all permissions below /tmp/test
chmod -R a+rwx /tmp/test Open all permissions to all below /tmp/test
```

The -R option is a handy feature of the chmod command. With -R, you can **recursively
change permissions of all files and directories starting from a point in the file system**. 

### setting the umask？

### changing ownership

更改所有者和组可以分别用不同的命令，也可以用chown一次完成

```bash
chown chris test/ 
chown chris:market test/ //一次修改所有者和组
chgrp market test/
chown -R chris test/  //递归修改所有子目录文件
```

# Traversing the File system

```bash
以上都是切换到用户目录
$cd
$cd $HOME
$cd ~

$cd ~francois 切换到xx工作目录

$cd -
$cd $OLDPWD //切换到前一个工作目录

$cd ~/web
$cd ..

$cd /usr/bin 和下面的区别是/表示根目录下
$cd usr/bin 当前目录下
```

### 查看当前目录

```bash
$ pwd
/Users/myerman
```

1. Using the -P and -L options to pwd and cd, you can **work with symbolically linked directories in their permanent or link locations***,* respectively.

![Untitled](Chapter%204%20covers%20commands%20for%20working%20with%20files%2032371ac794b641b7b1aa2adf5562d48a/Untitled%201.png)

# Copying Files

## CP

拷贝文件的时候，可以通过不同的参数来实现是否采用新的时间标签或者是 保留原来的

```bash
cd ; mkdir public_html ; touch index.html # 
cp -i index.html public_html/ #创建新时间标签
cp -il index.html public_html/ #创建一个硬连接而不是copy
cp -Rv public_html/ /mnt/usb/ #递归拷贝子目录，v参数会根据用户更新权限和时间
```

## DD

The dd command is another way to **copy data**. This command is very powerful
because on Mac OS X, everything is a file, including hardware peripherals. Here is
an example:

```bash
$ dd if=/dev/zero of=/tmp/mynullfile count=1
1+0 records in
1+0 records out
512 bytes transferred in 0.000253 secs (2022113 bytes/sec)
```

> `/dev/zero` is a special file that generates null characters.
> 

### 利用dd来备份数据

需要先将盘符umount

```bash
umount /dev/da0s1
dd if=/dev/da0s1 | gzip > bootpart.gz
```

### 指定复制数据大小

指定数据块大小是2字节，复制10个

```bash
$ dd if=/dev/zero of=/tmp/mynullfile count=10 bs=2
10+0 records in
10+0 records out
20 bytes transferred in 0.000367 secs (54507 bytes/sec)
```

从第二个IDE硬盘复制MBR引导记录到文件

```bash
dd if=/dev/ad1s1 of=mymbrfile bs=512 count=1
```

### **利用dd_rescue可以制作ISO文件**

并可以获得更多的输出资讯

```bash
dd_rescue /dev/acd0 myimage.iso
```

# 查找文件Searching for Files

在搜索文件之前，系统需要创建搜索数据库文件`/etc/locate.rc`

查找文件的主要命令是  `locate` and `find` 

## 产生本地数据库

通过运行如下命令来产生本地数据库

```bash
$ /usr/libexec/locate.updatedb 
```