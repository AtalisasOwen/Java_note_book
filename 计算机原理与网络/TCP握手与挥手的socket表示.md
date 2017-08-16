# Socket

* ### socket库允许访问本地C套接字库，通过控制网络层协议与传输层协议来控制如何发送数据

```py
import socket
s = socket.socket(AF_INET, SOCK_STREAM)
```

| 网络层协议 | 传输层协议 |
| :---: | :---: |
| AF\_INET\(IPv4\) | SOCK\_STEAM\(TCP\) |
| AF\_INET\(IPv6\) | SOCK\_DGRAM\(UDP\) |
| AF\_UNIX\(使用UNIX操作系统\) |  |

```py
#客户端
import socket
host = "localhost"
port = 50007
with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:#打开socket
    s.connect((host,port))                                  #连接至服务器，开始握手
    s.sendall(b'Hello World')
    data = s.recv(1024)
                                                            #with结束，调用close()开始挥手
print("receive:",repr(data))

#服务端
import socket
host = ""
port = 50007
with socket.socket(socket.AF_INET, socket.SOCK_STREAM) as s:#打开socket
    s.bind((host,port))                                     #绑定端口
    s.listen(1)                                             #每次只处理一个连接请求
    conn, addr = s.accept()                                 #当客户端发送请求，开始握手
    with conn:
        print("Connected by ",addr)
        while True:
            data = conn.recv(1024)
            if not data: break                             #客户端close()
            conn.sendall(data)
                                                           #内部with代码块，调用close()
```

### 建立TCP连接![](/assets/搜狗截图17年08月17日0227_1.png)![](/assets/搜狗截图17年08月17日0240_2.png)![](/assets/搜狗截图17年08月17日0241_3.png)![](/assets/搜狗截图17年08月17日0258_4.png)



