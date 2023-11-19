# Openwrt系统安装到物理机

主要方法：dd直接写盘

![Untitled](Openwrt%E7%B3%BB%E7%BB%9F%E5%AE%89%E8%A3%85%E5%88%B0%E7%89%A9%E7%90%86%E6%9C%BA%206ab8ca24a4bc4a069630a755d4484daf/Untitled.png)

### dd的解释

`dd if=/tmp/upload/op.img of = dev/sda`

if = Inbound/Input File

of= Outbound/output File

> 关于sda，表示系统中的第一个硬盘，如果是sda1则表示该硬盘的第一个分区，所以这里可以看出来是从img文件中读取数据直接写到第一个硬盘，而无视里面的分区。
> 

<aside>
💡 关于tmp：tmp其实是linux系统中在内存的一块区域，所以将文件传入tmp其实是传入到内存中。所以这样可以保证在写硬盘的时候，其实不会把tmp的内容写掉，因为tmp其实在内存。

</aside>