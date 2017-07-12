---
title: Turn and Stun server
---
本文简介了Turnserver（Turn + Stun）服务器的搭建。Turnserver主要提供了stun服务，支撑NAT、防火墙穿透，turn服务器，支撑打洞失败时的数据中转。使用场景上类似于前端使用的WEBRTC音视频数据服务，在不同网络环境下可通过stun服务器进行打洞以及turn服务器进行中转，最终实现web前端上的音视频通信。

### Turnserver 搭建
-----
#### 简介
webrtc的p2p穿透部分，一般都需要借助于turnserver，步骤大概是这样的
1.	尝试直连
2.	通过stun服务器进行NAT、防火墙穿透
3.	穿透失败则依赖于turn服务器中转

#### 一、安装turnserver（CentOS）
1. 下载安装依赖库（依赖于libevent，最好也安装下mysql）
```
yum install -y make automake gcc cc gcc-c++ wget #一般系统都带了gcc4，无需升级
yum install -y openssl-devel libevent libevent-devel mysql-devel mysql-server
wget https://github.com/downloads/libevent/libevent/libevent-2.0.21-stable.tar.gz

tar -zxvf libevent-2.0.21-stable.tar.gz #解压缩
cd libevent-2.0.21-stable.tar.gz
sudo ./configure
sudo make
sudo make install
cd ..
```
2. 下载turnServer安装包
这里使用都是4.4.5.2版本。也可以到[http://turnserver.open-sys.org/downloads/](http://turnserver.open-sys.org/downloads/) 下载你想要的版本。
```
wget http://turnserver.open-sys.org/downloads/v3.2.3.95/turnserver-4.4.5.2.tar.gz
tar -zxvf turnserver-4.4.5.2.tar.gz
cd turnserver-4.4.5.2.tar.gz
sudo ./configure
sudo make
sudo make install
cd ..
```
3. 接下来是配置turnserver.conf文件
可以到[your turnserver path]/examples/etc/下找到默认的turnserver.conf文件，建议拷贝到系统/etc/turnserver.conf。编辑该文件：
```cmd
# 配置为你服务器的外网ip地址
external-ip=106.74.23.11

# 由于WEBRTC默认需要使用long-term认证机制（中转服务器需要），所以必须添加-a(--lt-cred-mech)配置。使用该配置后，需要指定-r（realm）并且配置账号密码。修改turnserver.conf文件的
user=ghost:password
realm=demo

# 配置你的数据库连接，如果你想使用turn中转服务器，需要配置上一步的账号密码和realm值，只时候需要经由turnadmin工具往数据库添加用户记录。因此需要用到数据落地工具，WEBRTC提供了mysql、sqlite、mongoDB等方式，你只需要选一个你熟悉的方式即可。如以下的mysql配置：
mysql-userdb="host=localhost dbname=turnserver user=root password=root connect_timeout=30"

# 当然，你也可以修改turnServer默认监听的端口3478，以及需要监听的内网ip(内网ip可配置多个，全部注释时系统自动获取所有网卡ip)：
listening-port=3478
listening-ip=127.0.0.1
listening-ip=0.0.0.0

# 还有其他非常多的配置项，包括中转服务器线程数、中转服务器地址等等，有兴趣的可以详细阅读turnserver.conf配置文件
```
4. 创建并导入数据库
简单配置过turnserver.conf后，我们需要做的还有是导入数据库文件。否则启动时会提示连接数据库超时或找不到指定数据库。数据库结构文件在[your turnserver path]/turndb/下，例如mysql的schema.sql。
```
mysql -u root -p
******
$ create Database turnserver;
$ use turnserver

$ [copy schema.sql into here, and database tables will be created automatic.]
```
5. 新增用户到数据库
```
turnadmin -a -u root -r demo -p root -M "host=localhost dbname=turnserver user=root password=root connect_timeout=30"
```
6. 启动turnserver服务
```
turnserver -a -o #-o 后台运行
```
7.  配置客户端
```javascript
const servers = {
	iceServers: [
		{
			url: 'stun:106.74.23.11:3478'
		}, {
			url: 'turn:106.74.23.11:3478',
			username: 'root',
			credential: 'root'
		}
	]
}

let rtc = new RTCPeerConnection(servers);
```
8. 简单测试
两个手机同时连入移动4g网络，在打开turn服务器情况下可通过stun实现打洞并数据p2p；
其中一个手机切换到联通网络，stun服务器无法打通，转由turn服务器中转。