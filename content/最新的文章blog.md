Title: Nginx1.2.6的安装与配置
Tags: Wiki
Category: Linux
Date: 2014-1-17 13:21

==Nginx篇==
这里讲述的是Nginx在CentOS平台的编译安装极其相关详细内容。因为是编译安装，所以
这里虽然定义的是1.2.6,但是也适用于其它版本。
===Nginx的编译安装===
    ====相关模块的编译与安装====
{{{ class="brush bash"
#####安装libmcrypt
cd /usr/local/src
tar zxvf libmcrypt-2.5.8.tar.gz
cd libmcrypt-2.5.8
./configure
make
make install
#####安装cmake
cd /usr/local/src
tar zxvf cmake-2.8.11.2.tar.gz
cd cmake-2.8.11.2
./configure
make
make install
#####安装pcre
cd /usr/local/src
mkdir /usr/local/pcre
tar zxvf pcre-8.33.tar.gz
cd pcre-8.33 
./configure --prefix=/usr/local/pcre
make
make install
#####安装libunwind
cd /usr/local/src
tar zxvf libunwind-1.1.tar.gz
cd libunwind-1.1
./configure
make
make install
#####安装gperftools优化工具
cd /usr/local/src
tar zxvf gperftools-2.0.tar.gz
cd gperftools-2.0
./configure --enable-frame-pointers
make
make install
echo -e "/usr/local/lib" > /etc/ld.so.conf.d/usr_local_lib.conf
/sbin/ldconfig
    }}}
----
===添加用户和组===
{{{
groupadd www
useradd -g www www -s /bin/false
}}}
----
===Nginx1.2.6的编译安装===
{{{class="brush :bash"
    cd /usr/local/src
    tar zxvf nginx-1.2.6.tar.gz
    cd nginx-1.2.6
    ./configure --prefix=/usr/local/nginx --with-google_perftools_module \
    --without-http_memcached_module --user=www --group=www --with-http_stub_status_module \
    --with-openssl=/usr/ --with-pcre=/usr/local/src/pcre-8.33
    make
    make install
    mkdir /tmp/tcmalloc
    chmod  777 /tmp/tcmalloc -R
    /usr/local/nginx/sbin/nginx
    ####这里的etc/nginx在前面服务器大纲里面详细定义了的，以后凡是遇到这再也的配置文件不再多说了，直接去Packages里面etc里找
    cp /usr/local/src/etc/nginx  /etc/rc.d/init.d/nginx
    chmod 775 /etc/rc.d/init.d/nginx
    chkconfig nginx on
    /etc/rc.d/init.d/nginx restart
    }}}
----
===Nginx扩展模块的安装===
Nginx扩展模块代表这给Nginx增加新的功能，如图片缩放,FastDfs等，它的模块是非常丰富的，很多事情其实都可以用Nginx简单来实现，我们这里主要介绍生产服务器上面的在使用的扩展模块的安装，Nginx因为架构好，所以安装模块也都一样，也很简单，演示一个其它的也都如此安装即可。
----
    # `安装任何模块之前`: 应该先获取当前正在运行的Nginx进程的配置信息。方法为:`/usr/local/nginx/sbin/nginx -V`,执行此命令后你会看见`./configure`的回显,然后你只需要在此之后增加所需要的信息，如`--add-module=/usr/local/src/nginx-http-concat --with-http_image_filter_module`,重新编译即可.
    # `安装任何模块之后`:需要执行Nginx的平滑升级。平滑升级简单理解就是Nginx二进制可执行文件发生变化后，在不影响服务的前提下切换为新版本的Nginx服务。
        {{{class="brush :bash"
            #1.备份旧的Nginx可执行文件,复制编译生成的Nginx执行文件
            mv /usr/local/nginx/sbin/nginx /usr/local/nginx/sbin/nginx.old
            cp objs/nginx /usr/local/nginx/sbin/    #objs是在Nginx源码文件下
            #2.测试新版本的Nginx是否正常
            /usr/local/nginx/sbin/nginx -t -c /usr/local/nginx/conf/nginx.conf
            #3.平滑升级Nginx
            kill -USR2 `cat /usr/local/nginx/logs/nginx.pid`
            #此时旧版本的Nginx pid变为oldbin，这是旧版本和新版本的Nginx同时运行，过一段时间等旧Nginx处理完用户请求后，执行下面操作
            #4.从容关闭旧版本的Nginx工作进程
            kill -WINCH `cat /usr/local/nginx/logs/nginx.pid.oldbin`
            #5.决定是否升级到新的Nginx
            kill -HUP `cat /usr/local/nginx/logs/nginx.pid.oldbin`
            kill -QUIT `cat /usr/local/nginx/logs/nginx.pid.oldbin`
            }}}
----
====图片缩放模块的安装====
{{{class="brush :bash"
    cd /usr/local/src/
    cd nginx-1.2.6
    ./configure --prefix=/usr/local/nginx --with-google_perftools_module --without-http_memcached_module \
    --user=www --group=www --with-http_stub_status_module --with-openssl=/usr/ \
    --with-pcre=/usr/local/src/pcre-8.33 --with-http_image_filter_module
    make
    }}}
----
====Http-concat模块的安装====
此模块是用来减少一个页面静态文件的请求数目，如一个页面有a.js、b.js、a.css、b.css、c.css两个js文件和3个css文件，如果不使用该模块，则每次都会向服务器发送5个http请求，而用http-concat合并后就是需要发送2个请求了，是一向很好的性能优化.
简单的用法示例:
{{{
    http://example.com/??style1.css,style2.css,foo/style3.css?v=102234

## Configuration example

    location /static/css/ {
                concat on;
                concat_max_files 20;
             }
    location /static/js/ {
                concat on;
                concat_max_files 30;
             }
    }}}
详细说明请查看该模块的ReadMe.md文件，这个模块在生产上虽然编译了，但在项目配置中没有启用.
----
{{{class="brush :bash"
        cd /usr/local/src/
        cd nginx-1.2.6
        ./configure --prefix=/usr/local/nginx --with-google_perftools_module --without-http_memcached_module \
        --user=www --group=www --with-http_stub_status_module --with-openssl=/usr/ \
        --with-pcre=/usr/local/src/pcre-8.33 --with-http_image_filter_module --add-module=/usr/local/src/nginx-http-concat
        make
    }}}
----
====fastdfs-nginx-module安装====
请参看
[[fastdfs|FastDfs4.0.6的配置与安装]]
----
===Nginx配置文件详解===
这个是Nginx篇最重要的地方，如何使用好Nginx也就完全在于这里了。所以需要首先明白一些Nginx配置文件的基本语法和相关内容。这里肯定不可能介绍全，但基本上服务器上面所使用的和Nginx配置的最基础内容是会在这里进行详细介绍的。
{{{
    注意:本文中所有使用的配置文件，即Nginx所有配置文件都会打包放在概览中约定的Packages/etc/nginx-conf.tar.gz
    }}}
----
====Nginx配置基本语法知识====
# 组成Nginx配置的基本元素。这里的元素在Nginx官方的描述里面称为Context.主要Context有:`events`、`http`、`server`、`location`.
# 主要指令: `include`、`set`、`if`、`rewrite`、`break`、`root`、`index`、`alias`、`last`。这是几个核心和常用的指令。每个指令的具体用法也都很简单，具体可以参考Nginx官网或者google一下，这里就不详细叙述了。
# location的基本执行流程:
    - location匹配的原型是这样的：location [=|~|~*|^~|@] /uri/ { … }
        - `=`是精确匹配.
        - `@`是命名的location，在正常的location匹配中不会使用，仅仅在内部跳转中才会使用到。
        - `~`是区分大小写的匹配.
        - `~*`是不区分大小写的匹配
        - `^~`表示中止正则匹配
    - 在一个请求中，匹配的顺序是这样的:
        - 带`=`前缀的先进行匹配，如果找到了，中止查找。
        - 所有其它location进行非正则的匹配，找到最精确匹配的那个，如果匹配到带`^~`前缀的，则中止查找。
        - 正则查找，按照我们配置文件中配置的location顺序进行查找。
        - 如果正则查找匹配成功，则使用此正则匹配的location，否则，使用第二步查找的结果。
    - 特别说明下`=`与`^~`的区别：`=`在匹配时，则匹配带`=`的location。而`^~`，则会匹配所有非`=`的非正则location，只有在确认它是最精确匹配的location后，才生效。
# Nginx内置变量:
    - $args, 请求中的参数;
    - $content_length, HTTP请求信息里的"Content-Length";
    - $content_type, 请求信息里的"Content-Type";
    - $document_root, 针对当前请求的根路径设置值;
    - $document_uri   与uri相同;
    - $host, 请求信息中的"Host"，如果请求中没有Host行，则等于设置的服务器名;
    - $limit_rate, 对连接速率的限制;
    - $request_method, 请求的方法，比如"GET"、"POST"等;
    - $remote_addr, 客户端地址;
    - $remote_port, 客户端端口号;
    - $remote_user, 客户端用户名，认证用;
    - $request_filename, 当前请求的文件路径名
    - $request_body_file, ??
    - $request_uri, 请求的URI，带参数;
    - $query_string, args相同;
    - $scheme, 所用的协议，比如http或者是https，比如rewrite  ^(.+)-   - scheme://example.com- 1  redirect;
    - $server_protocol, 请求的协议版本，"HTTP/1.0"或"HTTP/1.1";
    - $server_addr, 服务器地址，如果没有用listen指明服务器地址，使用这个变量将发起一次系统调用以取得地址(造成资源浪费);
    - $server_name, 请求到达的服务器名;
    - $server_port, 请求到达的服务器端口号;
    - $uri, 请求的URI，可能和最初的值有不同，比如经过重定向之类的
# 生产服务器上Nginx基本目录结构:[[|{{../../../public/images/nginxpath.png}}]]
----
====Mdaxue-Nginx-Config详解====
这里对Nginx配置文件核心进行着行的解释，基本流程按照上面介绍的Context顺序解析。
# 先来看`conf/nginx.conf`，这里是入口点。
{{{
user  coolnet coolnet; #指定Nginx工作进程使用的用户和组，如果是做Web服务器这个用户必须权限分配好
worker_processes  8;    #指定8个子进程
error_log  logs/error.log;  #错误日志
#error_log  logs/error.log  notice; #错误日志级别notice，下同
#error_log  logs/error.log  info;
#error_log  logs/debug.log  debug;
pid        logs/nginx.pid;  #进程pid文件，这里必须配置上面介绍Nginx平滑升级的时候有使用
google_perftools_profiles /tmp/tcmalloc;    #google内存优化工具
events {
    use epoll;
    worker_connections  65535;
}
#include rtmp.conf;  #加入了一个rtmp服务器，这个在生产上没有使用，可以注释掉
http {  #http请求开始
    include       mime.types;   #文件类型列表
    default_type  application/octet-stream; #默认的请求头类型
    #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
    #                  '$status $body_bytes_sent "$http_referer" '
    #                  '"$http_user_agent" "$http_x_forwarded_for"';

    #access_log  logs/access.log  main;
    #防DDOS攻击，勿删除
    #resolver 192.168.1.1;   #这个生产上面没有，代表Nginx使用那个DNSserver解析，这在有时候是很有用的
    #lua_code_cache off; #lua缓存，开启后会损失性能，但修改lua脚本会即时生效，off时需要reload Nginx，所以生产off，开发on
    #lua_package_path "/usr/local/openresty/lualib/resty/?.lua;;";   #lua相关包路径
    #access_by_lua_file "lua/test.lua"; #引入lua脚本，这个前期服务器上面没有使用，后期可能会部署。
    limit_conn_zone $binary_remote_addr zone=ddos:10m;  #建立一个连接限制区域，以供全局使用，这个显示是没10M连接一次
    limit_req_zone $binary_remote_addr zone=oneddos:10m rate=2r/s;# 这个建立了一个请求显示区域，没秒2个请求，以上两想是为了放DDos
    server_names_hash_bucket_size 128;
    client_header_buffer_size 32k;
    large_client_header_buffers 4 32k;
    client_max_body_size 300m;
    sendfile        on;
    tcp_nopush     on;
    fastcgi_connect_timeout 300;
    fastcgi_send_timeout 300;
    fastcgi_read_timeout 300;
    fastcgi_buffer_size 64k;
    fastcgi_buffers 4 64k;
    fastcgi_busy_buffers_size 128k;
    fastcgi_temp_file_write_size 128k;
    keepalive_timeout  0;
    #keepalive_timeout  60;
    tcp_nodelay on;
    server_tokens off;

    gzip  on;
    gzip_min_length  1k;
    gzip_buffers     4 16k;
    gzip_http_version 1.1;
    gzip_comp_level 2;
    gzip_types       text/plain application/x-javascript text/css application/xml;
    gzip_vary on;

    server  #代表虚拟主机的开始
        {
     listen       80;   #该虚拟主机监听的端口
     #server_name localhost;    #注释掉的话即为默认主机
     #include  rewrite.conf;
     index index.php index.html index.htm default.html default.htm default.php; #index指定，默认首页
     root html; #root指令网站的绝对路径
     access_log logs/192.168.1.88.log;  #访问日志
	error_log logs/192.168.1.88.err;    #错误日志
     location ~ .*\.(php|php5)?$    #php文件的处理
                        {
                                fastcgi_pass  unix:/tmp/php-cgi.sock;   #以Unix套接字的方式和php进行通讯
                                #fastcgi_pass  127.0.0.1:9000;  #以TCP/IP的方式和php进行通讯
    #上面前者的性能要高于后者，但后者可以跨服务器，因为是基于网络的，前者只能只本地php,具体哪种方式取决与环境
                                fastcgi_index index.php;
                                include fcgi.conf;  #fcgi配置文件
                        }
                location /status {
                        stub_status on;
                        access_log   off;
                }

                location ~ .*\.(gif|jpg|jpeg|png|bmp|swf)$
                        {
                                expires      30d;   #图片文件缓存30天
                        }
                location ~ .*\.(js|css)?$   #css/js静态文件缓存12小时
                        {
                                expires      12h;
                        }
		location /M00/ {    #fastdfs配置，详细的了接请看[[fastdfs|FastDfs的安装与配置]]
			alias /data/fastdfs/store/data/;
			ngx_fastdfs_module;
		}
                access_log off;     #关闭该匹配类型下的日志记录
        }

include  vhost/*.conf;  #引入虚拟主机文件
}
    }}}
----
我这里看到了我们刚刚说到的Nginx的基本执行流程,也就是Context的执行流程，即`http`->`server`->`location`。我们最后也看到了文档最后一行`include`引入了 `vhost/*.conf`文件,这就是我们所有虚拟主机文件。虚拟主机文件的命名规则是:`域名`+`.conf`,如biz.mdaxue.com.conf,我们这里就以这个文件为例，因为大多数虚拟主机配置都几乎是一样的，有不同的在下面会分别列出来解释。
`biz.mdaxue.com.conf`的内容为:(以后新建虚拟主机可以直接cp过去，按照下面的解释更改即可)
{{{
server  #虚拟主机开始
        {
                listen       80;
server_name biz.mdaxue.com;     #主机域名
access_log  logs/biz.mdaxue.com.log;    #访问日志存放路径
error_log  logs/biz.mdaxue.com.err;     #错误日志存放路径
index index.php index.html index.htm default.html default.htm default.php;  #默认文件列表
include crash.filter;   #加载公共放攻击过滤器,这是一些安全配置
root  /home/coolnet/projects/business;  #网站根目录
location ~ .*\.(php|php5)?$     #php配置支持
{
fastcgi_pass  unix:/tmp/php-cgi.sock;
fastcgi_index index.php;
include fcgi.conf;
}
location /status {
stub_status on;
access_log   off;
}

location / {    #该网站的转发规则
rewrite ^/(\w+)/(\w+)\.html?(.*)$ /index.php?c=$1&act=$2&$3 last;
rewrite ^/(\w+)$ /index.php?college=$1 last;
}		
include static.conf;    #加载静态文件处理配置
}
}}}
----
好，到此我们看到了一个完全的虚拟主机是怎么建立的，而且通过上面我们可以知道，这是一个模板，而且我们约定所有大学网都基本上是这样一个模式建立的虚拟主机，所以我们看这个配置文件，都加载了两个全局配置文件`crash.filter`、`static.conf`，这两个是全局的，所以如果以后有什么规则需要全站都使用的可以放到这里面来。接下来我们就来看看这两个文件:
# `crash.filter`:
    {{{
#在搜索引擎中打开分站都也和www.mdaxue.com
#if ($request_uri ~* .*/$){
 # return 405; 
 # set $on index;
#}
#防各种蜘蛛
#if ($http_user_agent ~* "360Spider|JikeSpider|Spider|spider|bot|Bot|2345Explorer|curl|wget|webZIP|qihoobot|Baiduspider|Googlebot|Googlebot-Mobile|Googlebot-Image|Mediapartners-Google|Adsbot-Google|Feedfetcher-Google|Yahoo! Slurp|Yahoo! Slurp China|YoudaoBot|Sosospider|Sogou spider|Sogou web spider|MSNBot|ia_archiver|Tomato Bot|NSPlayer|bingbot")
 #        {
#	 set $on "${on}spider";
#          return 403;
 #         }
#if ($on != "indexspider"){
# return 405;
#}
#防空头攻击
if ($http_user_agent ~ ^$){
	return 503;
}

#听力死链接
location ~* ^(.*)(studyhard/listen/html/English)(.*)$ {
	return 404;
}
#防盗链
#location ~ .*\.(jpeg|wma|wmv|asf|mp3|mmf|zip|rar|jpg|gif|png|swf|flv)$ {
 #   valid_referers none blocked www.mdaxue.com *.mdaxue.com;
  #   if ($invalid_referer) {
   #  return 403;
    #  }
#}

#limit_conn ddos 10;
#limit_req zone=oneddos burst=50;

#我们看到这里都注释掉了，是因为前期推广的原因，不需要屏蔽蜘蛛。这里的各项都可以按需求开启，以后有关安全限制的都可以放到这里来加载，那么全站就都可以生效了.
        }}}
# `static.conf`:
{{{
#server static files direct
	location ~* /group1/M00/.*-(\d+)-(\d+)\.(gif|jpe?g|png|bmp|swf)$ {      #这里是image_filter结合FastDfs作的图片缩放
		#alias /data/fastdfs/store/data/;   #alias指令指定该Context的文件配置路径
              #ngx_fastdfs_module;  #在此启用FastDfs模块组件
	      set $img_width $1;    #将location里面的正则$1赋值给img_width变量
	      set $img_height $2;   #将location里面的正则$2赋值给img_height变量
	      rewrite ^/(.*)-\d+-\d+\.(png|jpe?g|gif|ico)$ /$1.$2 break;    #将满足前面正则的url转发到实际可访问的路径
	      image_filter resize $img_width $img_height;   #image_filter 按指定宽高缩放图片
	      image_filter_buffer 2M;   #缩放图片时使用的buffer大小
              expires 1y;
              access_log off;
              gzip off;
        }
        location ~* ([0-9]+)-([0-9]+)\.(png|jpe?g|gif|ico)$ {   #这个也一样只是规则不一样
		set $img_width $1;
	      set $img_height $2;
	      rewrite ^(.*)-[0-9]+-[0-9]+\.(png|jpe?g|gif|ico)$ /$1.$2 break;
	      image_filter resize $img_width $img_height;
	      image_filter_buffer 2M;
              expires 1y;
              access_log off;
              gzip off;
        }
	location ~* ([0-9]+)x([0-9]+)(\.JPG|jpg|GIF|gif|png|PNG|ico|ICO|bmp|BMP)$ { #这个是用lua做的图片缩放，能满足所有功能并且支持缓存，性能更好,
    但暂时还没有部署到生产上面去
	
		set $img_width $1;
                set $img_height $2;
		set $ext $3;
		access_by_lua_file 'lua/thumb.lua';
		expires 1y;
                gzip off;
        }
	location ~* /group1/M00/ {  #全站启用fastDfs，这块很重要，必须开启
		alias /data/fastdfs/store/data/;
		ngx_fastdfs_module;
	}
        location ~* \.(mp3|doc|rar|zip|wmv|flv|avi|mpeg)$ {     #对以上这些定义的文件启用缓存，关闭日志，时间为1年
                expires 1y;
                access_log off;
                gzip off;
        }
	location ~* \.(png|jpe?g|gif|ico)$ {    #对全站图片启用缓存，关闭日志，时间为1天
                expires 1d; 
                access_log off;
        }   
        location ~* \.(css)$ {      #对全站css文件启用缓存，关闭日志，时间为1天
                expires 1d;
                access_log off;
        }
        location ~* \.(js)$ {   #对全站Js文件启用混存，关闭日志，时间为1小时
                expires 1h;
                access_log off;
        }
    }}}
----
最后我们来说明下最复杂的一个配置文件:`www.mdaxue.com.conf`
{{{
    upstream www.mdaxue.com{    #这里定义了一个反向代理,名称为www.mdaxue.com
        server 127.0.0.1:8080;  #该反向代理转发的服务器
	    ip_hash;    #负载均衡时有用，代表一个Ip对应一台服务器，这个代表轮循规则，使用这种方式可以实现Session共享，其它的几种方式可以查看相关文档
    }
	upstream home.mdaxue.com{
	    server 127.0.0.1;
	}
    server {
        listen 80;
        server_name www.mdaxue.com mdaxue.com;  #监听www.mdaxue.com和*.mdaxue.com虚拟主机
        if ( $host != "www.mdaxue.com" ) {      #如果主机不为www.mdaxue.com
            rewrite ^/(.*)$ http://www.mdaxue.com/$1 permanent;就交有上面定义的www.mdaxue.com反向代理处理，即Java部分
        }
        #上面这个if代表的是所有以www.mdaxue.com为域名并且url后面海有东西的交由Tomcat来处理，即上面定义的127.0.0.1:8080
        access_log  logs/www.mdaxue.com.log;
        root /home/coolnet/java/tomcat6/webapps/ROOT;
        include static.conf;
        location ~* ^/$ {   #这个代表网址刚好是www.mdaxue.com即总站首页，交由php来处理，即home.mdaxue.com这个虚拟主机对应的项目
            proxy_pass http://home.mdaxue.com;
            proxy_set_header Host home.mdaxue.com;
            proxy_redirect off;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            client_max_body_size 10m;
            client_body_buffer_size 128k;
            proxy_connect_timeout 90;
            proxy_send_timeout 90;

            proxy_buffer_size 4k;
            proxy_buffers 4 32k;
            proxy_busy_buffers_size 64k;
            proxy_temp_file_write_size 64k;

        }
        location / {    #这个默认所有分站走这里面
            rewrite /$ /default.do;
            include www.mdaxue.com.rewrite.conf;    #加载www.mdaxue.com转发规则，该规则不在Nginx里面说明，详细说明请看Mdaxue项目手册
            include crash.filter;		
            proxy_pass http://www.mdaxue.com;
            proxy_set_header Host $host;
            proxy_redirect off;
            proxy_set_header X-Real-IP $remote_addr;
            proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
            client_max_body_size 10m;
            client_body_buffer_size 128k;
            proxy_connect_timeout 90;
            proxy_send_timeout 90;

            proxy_buffer_size 4k;
            proxy_buffers 4 32k;
            proxy_busy_buffers_size 64k;
            proxy_temp_file_write_size 64k;
        }

    }
    }}}
----
至此Nginx即完全说明完了，如有不足的可以后期修改和增加.
----
