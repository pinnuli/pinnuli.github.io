---
title: nginx 配置实现端口转发
date: 2018-04-04 22:14:46
categories: "服务器"
tags: 
    - nginx
---
最近在部署一个小程序的后台，但是小程序调用的接口是不能带端口号的，那么如果服务器上面80端口已经被其他程序占用，就只能采用端口转发或者虚拟目录，我采用的是端口转发，或者说当在一台主机上需要部署多个web应用，并且需要能在80端口访问这些web时，就可以采用这种方法，也可以叫做nginx反向代理用于实现负载均衡，这里记录一下遇到的一些小问题。

加入服务器域名是test.com,那么你可以通过test.com/news在80端口访问新闻应用，但是服务器上分配的是其他端口，如8081。
对应的nginx配置如下：

80端口的配置： 访问test.com/news => 127.0.0.1:8081 ,这里有一个需要注意的地方是转发的url最后需要加上'/'，
这相当指定了url'/',如果代理服务器地址中是带有URL的，此URL会替换掉 location 所匹配的URL部分, test.com/news/api,访问的是ip:8081/api;而如果代理服务器地址中是不带有URI的，则会用完整的请求URL来转发到代理服务器,test.com/news/api,访问的是ip:8081/news/api;

``` bash

server {
        listen       80;
 #      listen       [::]:80 default_server;
        server_name  test.com
        root         /usr/share/nginx/html;

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

        location / {
        }

        location /news{
                proxy_pass http:test.com:8081/;
        }

    }

```

8081端口的配置： 与平时配置没什么差别

``` bash

	server {
        listen 8081;
        server_name localhost;
        root /var/www/project;


        location / {
        index index.php index.html index.htm;
        if ( !-e $request_filename){
        rewrite ^(.*)$ /index.php?s=/$1 last;
        break;
                }
        }

	   #error_page 500 502 503 504  /50x.html;
       #location = /50x.html {
       #root /usr/share/ngixn/html;
       #}

       #我部署的是PHP项目，这里配置PHP解析
        location ~ \.php$ {
        fastcgi_pass 127.0.0.1:9000;
        fastcgi_index index.php;
        fastcgi_param SCRIPT_FILENAME $document_root$fastcgi_script_name;
        include /etc/nginx/fastcgi_params;
        include /etc/nginx/fastcgi.conf;
        }
}

```

