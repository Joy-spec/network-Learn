# socket 套接字编程
首先选择使用 TCP 套接字，还是使用 UDP 套接字
## UDP 套接字编程
### UDPClinet.py
```
from socket import *
serverName = '122.51.10.226'
serverPort = 12000
clientSocket = socket(AF_INET,SOCK_DGRAM)
message = raw_input('Input message to be send')
clientSocket.sendto(message,(serverName,serverPort))
receiveMessage,serverAddr = clientSocket.recvfrom(2048)
clientSocket.close()
```
### UDPServer.py
```
from socket import *
serverPort = 12000
serverSocket = socket(AF_INET,SOCK_DGRAM)
serverSocket.bind(('',serverPort))
while 1:
	message,clientAddr = serverSocket.recvfrom(2048)
	serverSocket.sendto('message send to client',clientAddr)
```
## TCP 套接字编程
### TCPClient.py
```
from socket import *
serverName = '122.51.10.226'
serverPort = 12000
clientSocket = socket(AF_INET,SOCK_STREAM)
clientSocket.connect((serverName,serverPort))
message = raw_input('Input message to be send')
clientSocket.sned(message)
receiveMessage = clientSocket.recv(2048)
clientSocket.close()
```
### TCPServer.py
```
from socket import *
serverPort = 80
serverSocket = socket(AF_INET,SOCK_STREAM)
serverSocket.bind(('',serverPort))
serverSocket.listen(1)
while 1:
	connectionSocket,addr = serverSocket.accept()
	sentence = connectionSocket.recv(1024)
	connectionSocket.send('the message to be send')
	connectionSocket.close()
```
## 对比
### client 与 server
- 客户端创建套接字时需要给出服务器的 IP 地址和端口号，至于客户端的套接字对应的端口号，可以由操作系统自动分配
- 服务器创建套接字时需要显式服务器的端口号，用于套接字绑定端口，而不是操作系统自动分配一个，因为客户端需要用到这个端口号
## TCP 对比 UDP
- 创建套接字时指定的第二个参数不一样，UDP 是SOCK_DGRAM，而TCP 是SOCK_STREAM
- TCP 的客户端创建套接字后，调用了connect,而 UDP 没有调用connect
- TCP 服务器首先创建了一个欢迎套接字（serverSocket)，这个套接字用于处理来自客户端的连接，它为每个连接创建一个连接套接字(connectionSocket)，  
与客户端的信息发送与接受是通过连接套接字进行的。而 UDP 没有为每一个连接创建一个单独的套接字，而是使用了同一个套接字
- TCP 与 UDP 使用了不同的发送与接受函数
### 原因分析
- TCP 是一个面向连接的协议，三次握手发生在客户套接字与服务端的欢迎套接字之间，  
三次握手之后欢迎套接字会生成一个连接套接字，连接套接字与客户端套接字建立一条TCP连接，如下面这条语句描述
```
connectionSocket,addr = serverSocket.accept()
```
- 因为 TCP 连接是个一对一连接，因此通过这个连接发送信息时应用层不需要再指定目的地址与端口（这些信息在建立连接的时候就已经确定了），  
所以看出通过 TCP 套接字接收和发送信息时使用的是 recv(messageLength) 和 send(message)，recv不需要指定目的地，recv也不会返回源地址
- UDP 连接不是一对一连接，它可以一对一，也可以一对多，因此需要在发送时指定目的地址和端口。所以 UDP 套接字接受和发送使用的函数是  
sendto(message,(serverIP,serverPort)),recvfrom(messageLength).前者指定了目的地，后者能够返回源地址（IP + 端口号）
