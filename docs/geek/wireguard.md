WireGuard是一种新型的虚拟专用网协议(VPN), 它运行在Linux内核层。WireGuard要求Linux内核版本`>=3.10`。

# 基本原理概览

WireGuard可以安全的封装UDP协议的IP包，一般是先创建一个网络接口，并给它配置一个私有密钥和一个要连接它的对端公钥，因为私钥公钥可以配对识别，所以可以在这两个端点之间进行安全的数据传递。

WireGuard可以创建一个或者多个接口，为每个接口配置自己的私钥和对端公钥，对端也可以是多个。配置过程可以和普通配置接口一样的逻辑，使用工具：`ifconfig`和`ip-address`，配置路由使用工具：`route`和`ip-route`以及一样其它常用的网络配置工具。对于WireGuard来说，可以使用一个特制的工具来配置网络：`wg`。配置的WireGuard接口可以作为一个隧道接口，安全交换数据。

WireGuard会给一个隧道设置IP地址、公钥和远程对端连接点。当使用隧道接口发送数据包到对端时，会进行下面的操作：

1. 假设IP数据包是从`192.168.30.8`的网络接口发出的，将要发送给对端网络接口`ABCDEFGH`，它会先在已经配置的所有对端接口中去查找匹配，如果没有匹配到，就把数据丢弃。
2. 如果找到了对端网络接口`ABCDEFGH`，就使用对端网络接口的公钥对要发送的IP数据包进行整体加密。因为使用了对端的公钥进行加密，只是使用对端的私钥才能正确解密，所以数据包只有到达了对端，才能被正确识别出来，这就保证了数据传输过程的安全性，即使数据包被其它接口截获也不能被解密。
3. 加密数据包后，要发送到对端`ABCDEFGH`时，先找到对端接口(`ABCDEFGH`)建立在主机`216.58.211.110`的`UDP`端口`53133`上面。
4. 把加密的数据包通过`UDP`协议发送到`216.58.211.110：53133`


当对端接口接收到加密数据包时，会进行下面的操作：

1. 从自己(216.58.211.110)的`UDP`端口`53133`获得了加密数据包后，开始使用自己的密钥进行解密
2. 从解密后的IP数据包中可以获得数据来自于`192.168.30.8`，然后查看自己的配置中是否允许接收来自这个源地址的数据。如果不允许，则把解密后的数据包丢弃。