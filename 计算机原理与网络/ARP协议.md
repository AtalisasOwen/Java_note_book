# ARP协议-地址解析协议

* ### 仅用于IPv4（IPv6使用邻居发现协议，被合并入ICMPv6）
* ### 用于解决地址问题，以目标IP地址为线索，用来定位下一个接收数据分包的网络设备对于的MAC地址
* ### 工作原理：源主机首先查找ARP缓存表，如果没有的话，会广播发送ARP请求包。如果IP地址不一样则忽略，一样就将自己的MAC地址写入ARP响应，告诉源主机。源主机接受到后，将IP和MAC地址写入ARP缓存表，并发送响应。



