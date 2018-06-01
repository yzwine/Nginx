Nginx反向代理配置以及部署
1.nginx介绍
（介绍地址：http://www.nginx.cn/nginxchswhyuseit）
nginx（发音"engine x"）是俄罗斯软件工程师Igor Sysoev开发的免费开源web服务器软件。nginx于2004年发布，聚焦于高性能，高并发和低内存消耗问题。并且具有多种web服务器功能特性：负载均衡，缓存，访问控制，带宽控制，以及高效整合各种应用的能力，这些特性使nginx很适合于现代网站架构。目前，nginx已经是互联网上第二流行的开源web服务器软件。

1.1.nginx主要功能
①.反向代理。
代理服务器将接收到的用户请求转发给内部服务器，再将内部服务器返回的结果返回给用户，此时代理服务器就充当一个服务器的角色。

②.负载均衡
将用户的请求均匀的或者按照一定的优先级分配到一组服务器中的一台上，而接收到请求的服务器独立的处理请求并返回。负载均衡技术主要用于扩展后端服务的性能。
2.下载nginx
下载地址：https://nginx.org/en/download.html

本次演示使用稳定版。
3.使用nginx
3.1.Nginx配置文件结构
下载下来的文件解压，打开nginx-1.12.2/conf/nginx.conf文件

...              #全局块

events {         #events块
   ...
}

http      #http块
{
    ...   #http全局块
    server        #server块
    { 
        ...       #server全局块
        location [PATTERN]   #location块
        {
            ...
        }
        location [PATTERN] 
        {
            ...
        }
    }
    server
    {
      ...
    }
    ...     #http全局块
}
1、全局块：配置影响nginx全局的指令。一般有运行nginx服务器的用户组，nginx进程pid存放路径，日志存放路径，配置文件引入，允许生成worker process数等。
2、events块：配置影响nginx服务器或与用户的网络连接。有每个进程的最大连接数，选取哪种事件驱动模型处理连接请求，是否允许同时接受多个网路连接，开启多个网络连接序列化等。
3、http块：可以嵌套多个server，配置代理，缓存，日志定义等绝大多数功能和第三方模块的配置。如文件引入，mime-type定义，日志自定义，是否使用sendfile传输文件，连接超时时间，单连接请求数等。
4、server块：配置虚拟主机的相关参数，一个http中可以有多个server。
5、location块：配置请求的路由，以及各种页面的处理情况。
3.2.反向代理案例讲解
原本从200.200.6.53服务器上获取数据，但出于安全性或者其他因素考虑，不直接从该服务器上获取数据，搭建一个代理服务器（192.168.1.1），访问代理服务器，代理服务器去原始服务器获取数据。

#①.访问192.168.1.1:8080/ssmDemo/getdata --> 200.200.6.53:8221/ssmDemo/getdata

	server {
        listen       8080;
        server_name  localhost;

        location / {
			#index index.jsp;
			#root   /root; 
			proxy_pass	 http://200.200.6.53:8221/;
			proxy_set_header	X-real-ip $remote_addr;
            proxy_set_header	X-Forwarded-For $proxy_add_x_forwarded_for;
			proxy_connect_timeout 5s; 
        }
}

②.访问192.168.1.1:8080/cip-cas/getdata --> 200.200.6.53:8443/cip-cas/getdata

···
	server {
        listen       8080;
        server_name  localhost;

        location /cip-cas {
			#index index.jsp;
			#root   /root; 
			proxy_pass	 http://200.200.6.53:8443/cip-cas/;
			proxy_set_header	X-real-ip $remote_addr;
            proxy_set_header	X-Forwarded-For $proxy_add_x_forwarded_for;
			proxy_connect_timeout 5s; 
        }
}
···
③.访问192.168.1.1:8080/Chapter1/getdata --> 200.200.6.53:9090/getdata

	server {
        listen       8080;
        server_name  localhost;

        location /Chapter [1-5] {
			#index index.jsp;
			#root   /root; 
			proxy_pass	 http://200.200.6.53:9090/;
			proxy_set_header	X-real-ip $remote_addr;
            proxy_set_header	X-Forwarded-For $proxy_add_x_forwarded_for;
			proxy_connect_timeout 5s; 
        }
}

(配置文件详解：https://www.cnblogs.com/hunttown/p/5759959.html )


proxy_set_header：就是可设置请求头-并将头信息传递到服务器端。不属于请求头的参数中也需要传递时 重定义下就行啦。

以下配置用于获取客户端（访问者）的ip地址：
1.proxy_pass	 http://200.200.6.53:9090/;
请求转向的服务器，如上转入的服务器地址为http://200.200.6.53:9090/，也可用列表的形式。

2.proxy_set_header    X-real-ip $remote_addr;
将$remote_addr的值放进变量X-Real-IP中，此变量名可变，$remote_addr的值为客户端的ip，设置代理服务器的IP地址为客户端地址。

3.proxy_set_header            X-Forwarded-For $proxy_add_x_forwarded_for;
  	把客户端的真是ip地址发送给服务器。

4.proxy_connect_timeout 5s;
连接超时时间为5秒。
3.3.启动关闭nginx
①.Windows环境下
方法一：双击nginx.exe应用程序，即可开启nginx服务（关闭比较麻烦）

方法二：nginx命令开启
1、启动：
C:\server\nginx-1.0.2>start nginx
或
C:\server\nginx-1.0.2>nginx.exe
2、停止：
C:\server\nginx-1.0.2>nginx.exe -s stop
或
C:\server\nginx-1.0.2>nginx.exe -s quit
注：stop是快速停止nginx，可能并不保存相关信息；quit是完整有序的停止nginx，并保存相关信息。
3、重新载入Nginx：
C:\server\nginx-1.0.2>nginx.exe -s reload
当配置信息修改，需要重新载入这些配置时使用此命令。
②.Linux环境下
1>.下载nginx（可能涉及到权限，用：chown cmreadwh 修改用户权限）
Linux下载先需要安装nginx应用程序
./configure
make
(sudo) make install
错误提示：
一、若出现该错误（./configure: error: the HTTP gzip module requires the zlib library.）
则需要安装“zlib-devel”，运行以下命令即可
yum install -y zlib-devel
二、若出现该错误（./configure: error: the HTTP rewrite module requires the PCRE library.）
则需要安装“pcre-deve”，运行以下命令即可
yum -y install pcre-devel



2>.修改nginx下的配置

3>.启动关闭nginx
1、启动：./sbin/nginx
2、停止：./sbin/nginx-s stop
3、重新载入Nginx：./sbin/nginx -s reload
3.4.nginx启动成功失败相关讲解
打开localhost，跳出nginx页面则启动成功，如下：

如打不开网页则配置有问题。

或看log下的错误日志


根据错误日志，修改配置，实现反向代理功能。
4.参考文献
https://www.cnblogs.com/knowledgesea/p/5175711.html
https://www.cnblogs.com/zhouxinfei/p/7862285.html
https://www.cnblogs.com/Miss-mickey/p/6734831.html
https://www.cnblogs.com/sixiweb/p/3988805.html
http://www.nginx.cn/nginxchswhyuseit
