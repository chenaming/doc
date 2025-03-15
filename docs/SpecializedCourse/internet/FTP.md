ftp(File Transfer Protocol)
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/f639a7b6365c4c55b055773dcb3d9a2b.png#pic_center)
- 向远程主机上传输文件或从远程主机接收文件
- 客户/服务器模式
	- 客户端：发起传输的一方
	- 服务器：远程主机
- ftp: RFC 959
- ftp服务器：端口号为21



FTP：控制连接与数据连接分开

![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/7edc2d7c59034b33b469073d90ed778f.png#pic_center)
- FTP客户端与FTP服务器通过端口21联系，并使用TCP为传输协议
- 客户端通过控制连接获得身份确认
- 客户端通过控制连接发送命令浏览远程目录
- 收到一个文件传输命令时，服务器打开一个到客户端的数据连接
- 一个文件传输完成后，服务器关闭连接
- 服务器打开第二个TCP数据连接用来传输另一个文件
- 控制连接：带外（"out of band"）传送
- FTP服务器维护用户的状态信息：当前路径、用户账户与控制连接对应（有状态）

FTP命令、响应
命令样例：
- 在控制连接上以ASCII文本方式传送
- USER username
- PASS password
- LIST：请服务器返回远程主机当前目录的文件列表
- RETR fileneme：从远程主机的当前目录检索文件（gets）
- STOR filename：向远程主机的当前目录存放文件（puts）

返回码样例：
- 状态码和状态信息（同HTTP）
- 331 Username OK, password required
- 125 data connection already open; transfer starting
- 425 Can't open data connection
- 452 Error waiting file

