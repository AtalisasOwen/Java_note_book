# socket选项

* ### setsocketopt\(level, optname, value\)
* ### getsocketopt\(level, optname\[, buflen\]\)

* ### level一般是SOL\_SOCKET, 表示使用socket选项

| optname选项名 | 意义 | 返回值 |
| :--- | :--- | :--- |
| SO\_BINDTODEVICE | 只在某个特殊网卡有效，可能不是移动设备 | 一个字符串返回设备名称，或默认为空 |
| SO\_BROADCAST | 允许广播地址发送和接受信息包，只对UDP有效 | 布尔值 |
| SO\_DONTROUTE | 禁止通过路由器和网关往外发送信息包，为了安全 | 布尔值 |
| SO\_KEEPALIVE | 可以使TCP通信的信息包保持连续性。 | 布尔值 |
| SO\_OOBINLINE | 把收到的不正常数据视为正常数据 | 布尔值 |
| SO\_REUSEADDR | 当socket被关闭后，占用的端口号能马上重用，不再保留几分钟 | 布尔值 |
| SO\_TYPE | 得到传输层协议，只用于get | 整数 |



