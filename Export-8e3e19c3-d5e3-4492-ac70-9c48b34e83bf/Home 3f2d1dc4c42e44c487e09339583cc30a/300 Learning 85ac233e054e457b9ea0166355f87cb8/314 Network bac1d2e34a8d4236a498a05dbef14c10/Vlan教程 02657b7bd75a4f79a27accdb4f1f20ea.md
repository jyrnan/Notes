# Vlan教程

[用最野的路子教你布局家庭网络@第二期--通过VLAN实现单网口软路由网口复用](https://www.youtube.com/watch?v=6RR12frrs-Y)

## Vlan种类

### 端口Vlan

路由器的Vlan是把路由器的网口分成不同的组，每个组内是同一个网络（广播域，也就是同一网段，网络值相同，不同主机名之间直接通过ip就可以互相访问）

### 802.1Q Vlan

也可以通过Taged方式设置成虚拟的隔离？

1、下面是定义的各种端口类型对各种数据帧的处理方法；

![Untitled](Vlan%E6%95%99%E7%A8%8B%2002657b7bd75a4f79a27accdb4f1f20ea/Untitled.png)

## 802.1Q Vlan小结：

### PVID和Tag

- PVID是端口默认的标签号，目的是给所有通过这个端口进入的无标签数据打上标签。
- untag的作用主要作用是在Access/truncked口上让数据离开网管交换机则不带标签（去掉和设置一致的ID标签）。每个端口必定有一个和PVID一致的unTagged标签
- Taged作用：一般用在Trunk端口，进出带标签流量必须符合Taged，Tag不同就不通过

以上二者结合起来可以实现Vlan，任意数据进入网管交换机就会实现分流。出去则带上某个标签或无标签

### 端口有两种类型

- Access 一般是终端数据进出，特征是必定有一个PVID和一致的unTaggedID
    
    ![Untitled](Vlan%E6%95%99%E7%A8%8B%2002657b7bd75a4f79a27accdb4f1f20ea/Untitled%201.png)
    
- Truncked 一般是用来连接其它交换设备，来自家其它交换机的进出流量。下图中Trunk口也可以设置一个新的tagged端口20，来保证路由器直接的数据都是带标签的。
    
    ![Untitled](Vlan%E6%95%99%E7%A8%8B%2002657b7bd75a4f79a27accdb4f1f20ea/Untitled%202.png)
    
- 也可以看到其实只有Trunk口是可能有多个标签流量，需要设置多个Tag，而Access口只接一个终端，所以只需要一对PVID和unTaggedID即可。
- 普通交换是无脑转发有无Tag的数据

## 范例

![Untitled](Vlan%E6%95%99%E7%A8%8B%2002657b7bd75a4f79a27accdb4f1f20ea/Untitled%203.png)

## 单臂路由器实例

![Untitled](Vlan%E6%95%99%E7%A8%8B%2002657b7bd75a4f79a27accdb4f1f20ea/Untitled%204.png)

![Untitled](Vlan%E6%95%99%E7%A8%8B%2002657b7bd75a4f79a27accdb4f1f20ea/Untitled%205.png)

![网管交换机和普通交换机连接范例](Vlan%E6%95%99%E7%A8%8B%2002657b7bd75a4f79a27accdb4f1f20ea/Untitled%206.png)

网管交换机和普通交换机连接范例

[RouterOS二层网桥上的VLAN配置](https://youtu.be/ysIF4-sc8pY)

- 配置端口pvid
- 配置网桥vlan，设置tag和untagged端口
- 启动bridge的vlan filtering，注意这里保持默认的pvid1

![IMG_7412.png](Vlan%E6%95%99%E7%A8%8B%2002657b7bd75a4f79a27accdb4f1f20ea/IMG_7412.png)

🤔

- 在bridge配置vlan是二层概念
- 在端口上去配置vlan是三层的概念

[Linux 中的虚拟网络接口](https://thiscute.world/posts/linux-virtual-network-interfaces/)

## **三、bridge**

Linux Bridge 是工作在链路层的网络交换机，由 Linux 内核模块 `brige` 提供，它负责在所有连接到它的接口之间转发链路层数据包。

添加到 Bridge 上的设备被设置为只接受二层数据帧并且转发所有收到的数据包到 Bridge 中。 在 Bridge 中会进行一个类似物理交换机的查MAC端口映射表、转发、更新MAC端口映射表这样的处理逻辑，从而数据包可以被转发到另一个接口/丢弃/广播/发往上层协议栈，由此 Bridge 实现了数据转发的功能。

如果使用 tcpdump 在 Bridge 接口上抓包，可以抓到网桥上所有接口进出的包，因为这些数据包都要通过网桥进行转发。

与物理交换机不同的是，Bridge 本身可以设置 IP 地址，可以认为当使用 `brctl addbr br0` 新建一个 br0 网桥时，系统自动创建了一个同名的隐藏 `br0` 网络接口。`br0` 一旦设置 IP 地址，就意味着这个隐藏的 br0 接口可以作为路由接口设备，参与 IP 层的路由选择(可以使用 `route -n` 查看最后一列 `Iface`)。因此只有当 `br0` 设置 `IP` 地址时，Bridge 才有可能将数据包发往上层协议栈。

但被添加到 Bridge 上的网卡是不能配置 IP 地址的，他们工作在数据链路层，对路由系统不可见。

它常被用于在虚拟机、主机上不同的 namepsaces 之间转发数据。

|  | eth1 | eth2 | eth3 | eth4 | eth5 | wlan1（2G） | wlan2（ 5G） |
| --- | --- | --- | --- | --- | --- | --- | --- |
|  |  |  |  | PVE | iMac |  |  |
| bridge | no | X | X |  |  |  |  |
| PVID | 1 | 1 | 1 | 1 | 10 |  |  |
| brigde Vlan 10 |  |  |  | tag | untag |  |  |

研发10.90.151.183

[https://dler.cloud/subscribe/L78bYjWrVWK9Lr8C0s2K?mu=smart&type=love](https://dler.cloud/subscribe/L78bYjWrVWK9Lr8C0s2K?mu=smart&type=love)

[https://dler.cloud/subscribe/L78bYjWrVWK9Lr8C0s2K?mu=ss&type=love](https://dler.cloud/subscribe/L78bYjWrVWK9Lr8C0s2K?mu=ss&type=love)