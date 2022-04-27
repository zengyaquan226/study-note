# Nginx的安装和启动

### 版本区别  

 常用版本分为四大阵营 

[Nginx开源版](http://nginx.org/) 				http://nginx.org/ 

[Nginx plus 商业版](https://www.nginx.com) 			https://www.nginx.com 

[openresty](http://openresty.org/cn/) 				http://openresty.org/cn/ 

[Tengine ](http://tengine.taobao.org/)					http://tengine.taobao.org/  

### Ngine的开源版的安装

.tar.gz放到linux操作系统 .. 目录下，使用tar zxvf安装

安装指令：

```java
 ./configure --prefix=/usr/local/nginx
```

安装到`usr/local/nginx`路径下

可能遇到的报错，需要安装的环境

 安装 `gcc yum install -y gcc`

 安装perl库 `yum install -y pcre pcre-devel  `

 安装zlib库 `yum install -y zlib zlib-devel  `

接下来执行 

```xml
make
make install
```

### 启动Ngine

进入安装好的目录 /usr/local/nginx/sbin  

需要的指令

```java
./nginx 启动
./nginx -s stop 快速停止
./nginx -s quit 优雅关闭，在退出前完成已经接受的连接请求
./nginx -s reload 重新加载配置
```

刚开始访问需要开启对应的防火墙

### 安装成系统服务，开机自启

 创建服务脚本 

```xml
vi /usr/lib/systemd/system/nginx.service  
```

服务脚本内容  

```xml
[Unit]
Description=nginx - web server
After=network.target remote-fs.target nss-lookup.target
[Service]
Type=forking
PIDFile=/usr/local/nginx/logs/nginx.pid
ExecStartPre=/usr/local/nginx/sbin/nginx -t -c /usr/local/nginx/conf/nginx.conf
ExecStart=/usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf
ExecReload=/usr/local/nginx/sbin/nginx -s reload
ExecStop=/usr/local/nginx/sbin/nginx -s stop
ExecQuit=/usr/local/nginx/sbin/nginx -s quit
PrivateTmp=true
[Install]
WantedBy=multi-user.target
```

 重新加载系统服务 

```
systemctl daemon-reload  
```



 启动服务 

```
systemctl start nginx.service  
```

使用系统启动服务之前，建议先把之前启动的关闭，否则可能有冲突



重新启动nginx服务

```
systemctl reload nginx
```



 开机启动  

```
systemctl enable nginx.service
```

 # 最小的配置文件的解析

```xml

#user  nobody;
#启动的时候工作的进程 配置的原则和cpu的数目一致即可
worker_processes  1;

#error_log  logs/error.log;
#error_log  logs/error.log  notice;
#error_log  logs/error.log  info;

#pid        logs/nginx.pid;

# 每一个工作的连接数可以为1024个
events {
    worker_connections  1024;
}


http {
# 引入这个文件 mine.type这个文件的就是告诉浏览器 以什么样子的方式来进行解析
    include       mime.types;
# 没有找到对应的将采用默认的
    default_type  application/octet-stream;

    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;

# on 代表没有一个复制的过程 nginx不需要自己将资源拷贝下来
    sendfile        on;
    #tcp_nopush     on;

    #keepalive_timeout  0;
    keepalive_timeout  65;

    #gzip  on;

# 相当于一个主机(虚拟主机)
    server {
# 在80端口进行监听 当有多个主机的时候 端口号和域名不可以完全相同
        listen       80;
#主机的每次 也可以为域名 
        server_name  localhost;

        #charset koi8-r;

        #access_log  logs/host.access.log  main;

# 这个相当于当我们发起 / 的请求的时候 去html(相对路径) 这个文件找 index.html index.htm
        location / {
            root   html;
            index  index.html index.htm;
        }

        #error_page  404              /404.html;

        # redirect server error pages to the static page /50x.html
        #
# 出现错误的时候去找资源
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

        # proxy the PHP scripts to Apache listening on 127.0.0.1:80
        #
        #location ~ \.php$ {
        #    proxy_pass   http://127.0.0.1;
        #}

        # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
        #
        #location ~ \.php$ {
        #    root           html;
        #    fastcgi_pass   127.0.0.1:9000;
        #    fastcgi_index  index.php;
        #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
        #    include        fastcgi_params;
        #}

        # deny access to .htaccess files, if Apache's document root
        # concurs with nginx's one
        #
        #location ~ /\.ht {
        #    deny  all;
        #}
    }


    # another virtual host using mix of IP-, name-, and port-based configuration
    #
    #server {
    #    listen       8000;
    #    listen       somename:8080;
    #    server_name  somename  alias  another.alias;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}


    # HTTPS server
    #
    #server {
    #    listen       443 ssl;
    #    server_name  localhost;

    #    ssl_certificate      cert.pem;
    #    ssl_certificate_key  cert.key;

    #    ssl_session_cache    shared:SSL:1m;
    #    ssl_session_timeout  5m;

    #    ssl_ciphers  HIGH:!aNULL:!MD5;
    #    ssl_prefer_server_ciphers  on;

    #    location / {
    #        root   html;
    #        index  index.html index.htm;
    #    }
    #}

}

```

# 虚拟主机

### 域名解析规则

-  **servername匹配规则**  

 我们需要注意的是servername匹配分先后顺序，写在前面的匹配上就不会继续往下匹配了。  



-  **完整匹配**  

 我们可以在同一servername中匹配多个域名  

```
server_name vod.mmban.com www1.mmban.com;
```



-  **通配符匹配**  

```
server_name *.mmban.com
```



-  **通配符结束匹配**  

```
server_name vod.*;
```



-  **正则匹配**  

```
server_name ~^[0-9]+\.mmban\.com$;
```

# 反向代理和正向代理

### 反向代理

  	描述:反向代理是指以代理服务器来接受连接请求，然后将请求转发给内部网络上的服务器，并将从服务器上得到的结果返回给请求连接的客户端，此时代理服务器对外就表现为一个反向代理服务器,而且整个过程对于客户端而言是透明的。 

***隧道式代理***：一进一出一个口

![image.png](https://cdn.nlark.com/yuque/0/2022/png/2755207/1650464505655-eb92c673-d4c5-41e4-8fed-0ae46ab54c30.png?x-oss-process=image%2Fresize%2Cw_750%2Climit_0)

### 正向代理

​		描述:正向代理意思是一个位于客户端和原始服务器(origin server)之间的服务器，为了从原始服务器取得内容，客户端向代理发送一个请求并指定目标(原始服务器)，然后由代理向原始服务器转交请求并将获得的内容返回给客户端。

![image.png](https://cdn.nlark.com/yuque/0/2022/png/2755207/1650464716867-64aa9a5b-5953-4eea-9e76-19f0b599aa44.png)

### 区别

​		无论是反向代理还是正向代理都是通过一个代理服务器进行中间进行通信，相当于网关，正向代理就是自己做一个代理服务器，反向代理就是服务器自己提供一个代理服务器进行来通信

### lvs（DR模型）

![image.png](https://cdn.nlark.com/yuque/0/2022/png/2755207/1650465758190-5296f379-692f-4253-a0f6-79449306e6ae.png)



***ivs于隧道模型相比，应用服务器返回信息的时候不再通过代理服务器，而是直接打到浏览器的网关***

### 反向代理的应用场景

![img](https://cdn.nlark.com/yuque/0/2022/png/2755207/1650466089930-2d9cb17e-27bd-464b-84f7-d1af5bac5171.png)

![image.png](https://cdn.nlark.com/yuque/0/2022/png/2755207/1650466122278-2d1a5744-a6c4-4d9f-86ad-b2895582d990.png?x-oss-process=image%2Fresize%2Cw_520%2Climit_0)

###  

### nginx的代理的配置

```Java
# 在nginx的配置文件中
server {
        listen       80;
        server_name  localhost;

        location / {
      # 进行代理 192.168.1.24 的服务器 所有对这个服务器的操作都要经过 nginx
	  proxy_pass http://192.168.1.24;
          #  root   html;
          #  index  index.html index.htm;
        }

   
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

}
```



# 负载均衡

### 负载均衡的基本配置

***在配置文件nginx.conf中***

```Java
# 采用upstream的方式进行和多个服务器进行反向代理
upstream httpds {
		server 192.168.1.23:80;
		server 192.168.1.24:80;
	}
	
    server {
        listen       80;
        server_name  localhost;

        location / {
      # httpds 指向上面的upstream
	  proxy_pass http://httpds;
          #  root   html;
          #  index  index.html index.htm;
        }
# 这个最简单的配置是雨露均沾 一人一次的来 （轮询）

```

#### 权重的配置（weight）

***权重就是在负载均衡的时候分的的比例的大小***

配置方式

```java
upstream httpds {
    # 大概的比例就是8：2：1但是是随机的不一定是这样的
		server 192.168.1.23:80 weight=8;
		server 192.168.1.24:80 weight=2;
		server 192.168.1.25:80 weight=1;
	}
	
    server {
        listen       80;
        server_name  localhost;

        location / {
      # 进行反向代理 
	  proxy_pass http://httpds;
          #  root   html;
          #  index  index.html index.htm;
        }

   
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

}
```

#### down 和 backup使用

1. down：***让这个服务器不参与负载均衡***
2. backup: ***将这个服务器作为备用机 当其他的服务器都挂了之后 使用***

配置方式：

```xml
upstream httpds {
	# down 不参与负载均衡
		server 192.168.1.23:80 weight=8 down;
		server 192.168.1.24:80 weight=2;
	# backup 当成备用机
		server 192.168.1.25:80 weight=1 backup;
	}
```



***注：***这个down和backup真实的使用不常见

### 负载均衡的ip_hash算法

​		ip_hash 会话粘连, 上面的2种方式都有一个问题，那就是下一个请求来的时候请求可能分发到另外一个服务器，当我们的程序不是无状态的时候（采用了session保存数据），这时候就有一个很大的很问题了，比如把登录信息保存到了session中，那么跳转到另外一台服务器的时候就需要重新登录了，所以很多时候我们需要一个客户只访问一个服务器，那么就需要用iphash了，iphash的每个请求按访问ip的hash结果分配，这样每个访客固定访问一个后端服务器，可以解决session的问题。 （但是ip可能会改变）

```java
# 会话粘粘可以理解为用户持续访问一个后端机器
upstream test {
  ip_hash;
  server weiyigeek.top:8080;
  server weiyigeek.top:8081;
} 
```



### 负载均衡的fair算法

​		fair（第三方）按后端服务器 的响应时间来分配请求，响应时间短的优先分配。

```xml
upstream backend {
  fair;
  server weiyigeek.top:8080;
  server weiyigeek.top:8081;
} 
```



### 负载均衡的url_hash算法

​		url_hash（第三方）:按访问url的hash结果来分配请求，使每个url定向到同一个后端服务器，后端服务器为缓存时比较有效。

​		在upstam中加入hash语句，server语句中不能写入weight等其他的参数，hash_method是使用的hash算法	 

```java
upstream backend {
  hash $request_uri;
  hash_method crc32;
  server weiyigeek.top:8080;
  server weiyigeek.top:8081;
} 
```



***注：***以上几种情况选择使用哪种策略模式,不过fair和url_hash需要安装第三方模块才能使用. 



# 动静分离

### 多个location进行匹配

***动静分离是指通过nginx反向代理到tomcat的服务器，然后静态的资源放到nginx上即可 无需去请求tomcat服务器***

实现步骤

```xml
upstream httpds {
# 代理到tomcat的服务器
		server 192.168.1.25:8080;
	}
server {
        listen       80;
        server_name  localhost;

        location / {
			proxy_pass http://httpds;
          #  root   html;
          #  index  index.html index.htm;
        } 
# 以/css/** 所有的请求都会来到这里 root指定的目录去找 html/css/** 去找 如果直接请求 / 找index
		location /css {
#css静态资源的位置 index 将相当于一个静态的访问页
           root   html;
           index  index.html index.htm;
        }

   
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

}
```



### 使用正则表达式进行匹配

* 不使用正则表达式的写法

  ```xml
   location / {
  			proxy_pass http://httpds;
            #  root   html;
            #  index  index.html index.htm;
          } 
  		
  location /css {
          root   html;
          index  index.html index.htm;
  }
  location /js {
          root   html;
          index  index.html index.htm;
  }
  location /img {
          root   html;
          index  index.html index.htm;
  }
  ```

* 使用正则表达式

  ```xml
  location ~*/(css|img|js) {
    root /usr/local/nginx/static;
    index index.html index.htm;
  }
  ```



# URLRewrite

***URLRewrite*** 隐藏真实的URL地址 用一个url地址进行代替



配置方式：

```xml
# 再配置文件中
server {
        listen       80;
        server_name  localhost;

        location / {
			rewrite ^/2.html$ /bookStroy/index.jsp break;
			proxy_pass http://httpds;
          #  root   html;
          #  index  index.html index.htm;
        } 
		
		location ~*/(css|js|img) {
           root   html;
           index  index.html index.htm;
        }

   
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

}
```

### URLRewire 的关键字

```java
rewrite是实现URL重写的关键指令，根据regex (正则表达式)部分内容，
重定向到replacement，结尾是flag标记。 

rewrite 	<regex> 	<replacement> 	[flag];
关键字 		正则 				替代内容 			flag标记

关键字：其中关键字error_log不能改变

正则：perl兼容正则表达式语句进行规则匹配 
替代内容：将正则匹配的内容替换成replacement 
flag标记：rewrite支持的flag标记 

rewrite参数的标签段位置： server,location,if 

flag标记说明： 
last			#本条规则匹配完成后，继续向下匹配新的location URI规则 
break 		#本条规则匹配完成即终止，不再匹配后面的任何规则
redirect 		#返回302临时重定向，浏览器地址会显示跳转后的URL地址   (防爬虫)
permanent 	#返回301永久重定向，浏览器地址栏会显示跳转后的URL地址  

例子：
   # 括号起到分组的作用 $1是外部的引用
 rewrite ^/([0-9]+).html$ /index.jsp?pageNum=$1 break;  


```



# 负载均衡+URLRewrite

***模拟真实的工作环境 外网无法访问tomcat这个服务器，只能通过nginx服务器进行反向代理***

![image.png](https://cdn.nlark.com/yuque/0/2022/png/2755207/1650508898486-e6aca3f1-cf8b-4c7f-8d88-d57432379c4d.png?x-oss-process=image%2Fresize%2Cw_576%2Climit_0)

### 一些防火墙的指令

打开防火墙

```
systemctl start firewalld
```

 

指定端口和ip访问

```
firewall-cmd --permanent --add-rich-rule="rule family="ipv4" source address="192.168.1.22" port protocol="tcp" port="8080" accept"
```

 

移除端口和ip访问

```
firewall-cmd --permanent --remove-rule="rule family="ipv4" source address="192.168.44.101" port protocol="tcp" port="8080" accept"
```



查看已配置规则  

```
firewall-cmd --list-all
```



重启防火墙

```
systemctl restart firewalld
```



重载规则

```
firewall-cmd --reload
```



# 网关服务器

***之前nginx服务器的功能有以下功能***

* 负载均衡
* 反向代理
* 动静分离
* URLRewrite

### 当前学习比较完整的nginx.conf配置文件

```

worker_processes  1;

events {
    worker_connections  1024;
}


http {
    include       mime.types;
    default_type  application/octet-stream;

    sendfile        on;
    keepalive_timeout  65;


# 需要反向代理到的服务器的地址
	upstream httpds {
	# down 不参与负载均衡 weight 权重
		server 192.168.1.23:80 weight=8;
		server 192.168.1.24:80 weight=2;
	# backup 当成备用机
		server 192.168.1.25:8080 weight=4;
	}
	
    server {
        listen       80;
        server_name  localhost;

        location / {
        # rewrite URLRewrite 
			rewrite ^/([0-9]+).html$ /bookStroy/index.jsp break;
			# 反向代理
			proxy_pass http://httpds;
          #  root   html;
          #  index  index.html index.htm;
        } 
		
		# 动静分离 
		location ~*/(css|js|img) {
           root   html;
           index  index.html index.htm;
        }

   
        error_page   500 502 503 504  /50x.html;
        location = /50x.html {
            root   html;
        }

}
}

```



# 防盗链

### **http协议中的 referer**

***浏览器会遵守http的协议，在第二次请求的时候带上 referrer在请求头里***

* 用于表明这一次的请求是是不是在html中引用的一些资源

可以用于判断是否是同一个站点 是否是其他的站点引用这个站点的资源



 ### 防盗链的配置方式

在需要配置防盗链的location路径进行配置即可

```
 # 192.168.1.22 这里在生产环境最好配置 域名
 # 这个的意思就是 如果请求头中的referer 的地址对应不上 或者没有 直接进行访问 将返回 状态码 403
 # 有none 就是可以直接访问 没有referer也可以
 valid_referers none 192.168.1.22; // 或者 valid_referers 192.168.1.22;
		   # if 后面需要空格
           if ($invalid_referer){
           return 403;
}
```



***防盗链的配置***

```
valid_referers none|blocked| server_names;
none:监测访问不带referer的情况 即就是第一次访问 或者直接访问
blocked:检测rederer的头是否被隐藏 即就是检测 http:// 和 https://
sercer_names:设置一个或多个URL，检测rederer的值是否是其中的一个
```



### 返回自定义的错误页面

***在server中加入***

```
# error_page   403  /403.html 不可以省略 不然不是自己定义的403页面
# 当之前的返回的状态码是403 时 转到 403.html中 在html的目录下
		error_page   403  /403.html;
        location = /403.html {
            root   html;
   }
```



### 返回提示图片

***在防盗链中的配置改成***

```
location ~*/(css|js|img) {
		# 进行防盗链的配置 即 其他站点不可以访问这个（192.168.1.22）这个站点的location里面的内容
		   valid_referers 192.168.1.22;
		   # if 后面需要空格
		   if ($invalid_referer){
		   # ^/ 相当于进来的所有请求 都去请求这个 html/error.png
				rewrite ^/ /error.png break;
				# 不进行return
				# return 403;
		   }
           root   html;
           index  index.html index.htm;
        }

```



# 高可用

![image.png](https://cdn.nlark.com/yuque/0/2022/png/2755207/1650521711859-a3f15f4e-e25d-45dc-a872-bae78e1d83ba.png?x-oss-process=image%2Fresize%2Cw_596%2Climit_0)



​		意思：当有两个nginx的时候一个作为主机，一个作为备用机，因为在局域网中，IP不可以相同，不能要请求不停的换ip来进行请求，这个时候可以使用keepaloved的进行虚拟的ip，让这个虚拟的ip在两个nginx服务器上不同的切换，当一台服务器挂了之后，另一个可以顶上，并且请求都是同一个ip。



### 安装Keepalived  

 下载地址  

https://www.keepalived.org/download.html#



 使用 ./configure 编译安装  

 如遇报错提示  

```
configure: error:
!!! OpenSSL is not properly installed on your system. !!!
!!! Can not include OpenSSL headers files. !!!
```

安装依赖  

```
 yum install openssl-devel  
```

 yum安装    直接使用这个就可以

```
 yum install -y  keepalived  
```



- 配置后

使用yum安装后，配置文件在

 ***/etc/keepalived/keepalived.conf***  



### 最小的配置文件

```
! Configuration File for keepalived

global_defs {
  
   router_id bb22

}

vrrp_instance VI_1 {
#主机
    state MASTER
# 通过 IP addr查出的网卡
    interface ens33
    virtual_router_id 51
    # 优先级 越高 机会越大
    priority 100
    advert_int 1
    # 用这个来确定是否是一个组 可以ip移动
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    # VIP 虚拟ip 可以设置多个
    virtual_ipaddress {
        192.168.1.21
    }
}

====================================================================================================
# 第二个的配置文件
! Configuration File for keepalived

global_defs {
  
   router_id bb21

}

vrrp_instance VI_1 {
    state BACKUP
    interface ens33
    virtual_router_id 51
    priority 50
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
        192.168.1.21
    }
}


```



# https

### 不安全的http协议

![img](https://cdn.nlark.com/yuque/0/2022/png/2755207/1650523871523-2b867c32-183b-4d38-a176-36baf0ea962f.png)



### ca机构参与保证互联网安全

![img](https://cdn.nlark.com/yuque/0/2022/png/2755207/1650525306351-05b799b9-eaae-4343-9534-0c94f21374c1.png)



### 证书安装

先使用域名申请证书。（可以去阿里云购买对应的证书）

将证书放到/usr/local/nginx/conf目录中。



ssl_certificate			xx.pem;

ssl_certificate_key		xx.key;



- 安装到nginx

![img](https://cdn.nlark.com/yuque/0/2022/png/2755207/1650528483196-94b44590-bab2-4fd0-a6d5-6f0cf8f9d510.png)



现在访问 server_name 对应的域名，即可安全



### 配置之后的所有的请求都直接到https去

```
server {
	listen 80;
	server_name 域名; # 已经注册了证书的域名
	return 301 https://$server_name$request_uri;
}
```

