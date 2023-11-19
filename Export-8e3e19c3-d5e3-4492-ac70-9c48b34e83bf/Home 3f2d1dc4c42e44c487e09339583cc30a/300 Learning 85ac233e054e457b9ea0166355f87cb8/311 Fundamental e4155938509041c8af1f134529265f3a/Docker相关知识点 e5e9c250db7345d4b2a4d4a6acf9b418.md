# Docker相关知识点

# Docker的网络类型

## Bridge

bridge模式是创建一个虚拟网卡veth0桥接到虚拟网桥br0，所以需要一个地址转换再链接到宿主的物理网卡，采用NAT方式通信

![Untitled](Docker%E7%9B%B8%E5%85%B3%E7%9F%A5%E8%AF%86%E7%82%B9%20e5e9c250db7345d4b2a4d4a6acf9b418/Untitled.png)

> docker默认的模式就是这种，一般分配的ip地址 `172.17.0.0/16`，通过NAT和宿主物理网卡通信
> 

## Macvlan

这种形式会给每个虚拟主机一个虚拟的mac地址，这样可以指定固定的ip地址和宿主物理网卡真正链接

![Untitled](Docker%E7%9B%B8%E5%85%B3%E7%9F%A5%E8%AF%86%E7%82%B9%20e5e9c250db7345d4b2a4d4a6acf9b418/Untitled%201.png)

<aside>
💡 需要注意：Macvlan模式下，宿主机和容器也不能直接通信，即使分配的Ip在同一网段也不行。但是同一个vlan中容器之间是可以通信的。（为什么？）

</aside>

### Macvlan的模式

1. Bridge Mode：一般采用这个
2. Private Mode
3. VEPA
4. Passthrough

### 创建Macvlan实操关键点

- 父接口：一般是宿主机的物理网络口？一般是有ip地址的那个网络接口

![Untitled](Docker%E7%9B%B8%E5%85%B3%E7%9F%A5%E8%AF%86%E7%82%B9%20e5e9c250db7345d4b2a4d4a6acf9b418/Untitled%202.png)

- 子网络：最好设置成主机同网段，例如 `192.168.6.1/24`
- 网关：必须不能和父接口的ip地址不一样。原因是[上面](Docker%E7%9B%B8%E5%85%B3%E7%9F%A5%E8%AF%86%E7%82%B9%20e5e9c250db7345d4b2a4d4a6acf9b418.md)。宿主机并不一定承担网关的角色。 一般是填写该网络的真正网关地址： 例如`192.168.5.1`
- IP范围：可以指定一个分配的ip范围，目的是不要和父网络里面其它机器的ip重合。例如填写 `192.168.5.33/28`（是一个子网的表达方式）。最好强制容器使用某IP

随后系统会生成一个虚拟的网络接口，macvlan_eNet，ip地址就是上面设置的网关  `192.168.5.1`

![Openwrt里增加的接口](Docker%E7%9B%B8%E5%85%B3%E7%9F%A5%E8%AF%86%E7%82%B9%20e5e9c250db7345d4b2a4d4a6acf9b418/Untitled%203.png)

Openwrt里增加的接口

防火墙已经将这个接口加入到lan内

![Untitled](Docker%E7%9B%B8%E5%85%B3%E7%9F%A5%E8%AF%86%E7%82%B9%20e5e9c250db7345d4b2a4d4a6acf9b418/Untitled%204.png)

# eSir安装docker旁路由方法

安装启动

![Untitled](Docker%E7%9B%B8%E5%85%B3%E7%9F%A5%E8%AF%86%E7%82%B9%20e5e9c250db7345d4b2a4d4a6acf9b418/Untitled%205.png)

安装启动容器后，进入容器命令行进行设置，界面是`ash`

![Untitled](Docker%E7%9B%B8%E5%85%B3%E7%9F%A5%E8%AF%86%E7%82%B9%20e5e9c250db7345d4b2a4d4a6acf9b418/Untitled%206.png)

编辑这个docker中的openwrt的ip地址 

命令是 `nano ./etc/config/network`

![Untitled](Docker%E7%9B%B8%E5%85%B3%E7%9F%A5%E8%AF%86%E7%82%B9%20e5e9c250db7345d4b2a4d4a6acf9b418/Untitled%207.png)

重新启动网络

![Untitled](Docker%E7%9B%B8%E5%85%B3%E7%9F%A5%E8%AF%86%E7%82%B9%20e5e9c250db7345d4b2a4d4a6acf9b418/Untitled%208.png)

这时候应该可以通过浏览器输入ip地址打开luci的网页界面了

登录后进入网络接口页面，可以干掉其它接口，仅保留lan口。ip地址应该是前面创建容器的时候设置的地址

![Untitled](Docker%E7%9B%B8%E5%85%B3%E7%9F%A5%E8%AF%86%E7%82%B9%20e5e9c250db7345d4b2a4d4a6acf9b418/Untitled%209.png)

设置lan网络的桥接属性为关闭（可以避免发送广播信息降低性能）并关闭DHCP功能

![Untitled](Docker%E7%9B%B8%E5%85%B3%E7%9F%A5%E8%AF%86%E7%82%B9%20e5e9c250db7345d4b2a4d4a6acf9b418/Untitled%2010.png)

最后设置lan的网关地址，指向主路由。

## Docker关闭和启动命令

![Untitled](Docker%E7%9B%B8%E5%85%B3%E7%9F%A5%E8%AF%86%E7%82%B9%20e5e9c250db7345d4b2a4d4a6acf9b418/Untitled%2011.png)