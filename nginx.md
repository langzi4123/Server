nginx相关知识

分析：nginx要成功响应请求，会有如下两个限制：

1、nginx接受的tcp连接多，能否建立起来?

2、nginx响应过程,要打开许多文件，能否打开?

所以，只要我们针对上面两个限制进行优化，就能大幅提升Nginx的效率。

系统层面
1、可打开句柄
	一般centos默认句柄打开数量是1024，使用 ulimit -n 可以查看服务器的默认句柄数量，修改服务器句柄数量，同时修改nginx配置保证nginx可以开启更多文件

2、TCP最大连接数（somaxconn）
	修改系统配置，例如 echo 10000 > /proc/sys/net/core/somaxconn

3、TCP连接立即回收、回用（recycle、reuse）
	echo 1 > /proc/sys/net/ipv4/tcp_tw_reuseecho 1 > /proc/sys/net/ipv4/tcp_tw_recycle

4、不做TCP洪水抵御（谨慎修改)
	echo 0 > /proc/sys/net/ipv4/tcp_syncookies

nginx层面

1： 优化 workprocess，cpu

2： 事件处理模型优化

nginx的连接处理机制在于不同的操作系统会采用不同的I/O模型，Linux下，nginx使用epoll的I/O多路复用模型，在freebsd使用kqueue的IO多路复用模型，在solaris使用/dev/pool方式的IO多路复用模型，在windows使用的icop等等。 要根据系统类型不同选择不同的事务处理模型，我们使用的是Centos，因此将nginx的事件处理模型调整为epoll模型。

events {
    worker_connections  10240;    // 
    use epoll;
}
说明：在不指定事件处理模型时，nginx默认会自动的选择最佳的事件处理模型服务。

3： 设置work_connections 连接数

4: 每个进程的最大文件打开数worker_rlimit_nofile 

5: keepalive timeout会话保持时间

6: GZIP压缩性能优化,如下

gzip on;       #表示开启压缩功能
gzip_min_length  1k; #表示允许压缩的页面最小字节数，页面字节数从header头的Content-Length中获取。默认值是0，表示不管页面多大都进行压缩，建议设置成大于1K。如果小于1K可能会越压越大
gzip_buffers     4 32k; #压缩缓存区大小
gzip_http_version 1.1; #压缩版本
gzip_comp_level 6; #压缩比率， 一般选择4-6，为了性能gzip_types text/css text/xml application/javascript;　　#指定压缩的类型 gzip_vary on;　#vary header支持

7：proxy超时设置 如下

proxy_connect_timeout 90;
proxy_send_timeout  90;
proxy_read_timeout  4k;
proxy_buffers 4 32k;
proxy_busy_buffers_size 64k

8：高效传输模式
sendfile on; # 开启高效文件传输模式。
tcp_nopush on; #需要在sendfile开启模式才有效，防止网路阻塞，积极的减少网络报文段的数量。将响应头和正文的开始部分一起发送，而不一个接一个的发送


根据我的经验，一般php服务器瓶颈基本不在nginx上，所以大多数时候保持这些修改就可以满足服务器绝大多数情况下的平稳运行了