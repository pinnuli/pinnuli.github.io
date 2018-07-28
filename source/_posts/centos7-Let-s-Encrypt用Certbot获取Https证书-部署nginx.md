title: centos7 Let's Encrypt用Certbot获取Https证书 部署nginx
date: 2018-07-15 17:24:42
tags:
---

- 检查nginx下是否有`--with-http_stub_status_module` 和`--with-http_ssl_module`两个模块，安装之后再重新编译
	> nginx -V

- 用http克隆github上的certbot
	>git clone https://github.com/certbot/certbot /opt/certbot-master
	
- 安装所有依赖
 	>/opt/certbot-master/letsencrypt-auto --help
- 关闭nginx，检出80端口，443端口是否有开启
	>nginx -s stop

	>firewall-cmd --query-port=80/tcp


	>firewall-cmd --query-port=443/tcp
	
	没有的话就开启
	>firewall-cmd --permanent --zone=public --add-port=80/tcp
- 获取证书
	>/opt/certbot-master/letsencrypt-auto --nginx -d www.pinnuli.com 
- 配置nginx（也可选择自动配置)
	
	>     user nginx;
	>     worker_processes auto;
	>     error_log /var/log/nginx/error.log;
	>     pid /run/nginx.pid;
	> 
	>     include /usr/share/nginx/modules/*.conf;
	> 
	>     events {
	>         worker_connections 1024;
	>     }
	> 
	>     http {
	>         log_format  main  '$remote_addr - $remote_user [$time_local] "$request" 
	>                      '$status $body_bytes_sent "$http_referer" '
	>                       '"$http_user_agent" "$http_x_forwarded_for"';
	> 
	>         access_log  /var/log/nginx/access.log  main;
	> 
	>         sendfile            on;
	>         tcp_nopush          on;
	>         tcp_nodelay         on;
	>         keepalive_timeout   65;
	>         types_hash_max_size 2048;
	> 
	>         include             /etc/nginx/mime.types;
	>         default_type        application/octet-stream;
	> 
	>         include /etc/nginx/conf.d/*.conf;
	> 
	> 	      server {
	>             listen       80 default_server;
	>             listen       [::]:80 default_server;
	>             server_name  _;
	>             root        /var/www/pinnuli.github.io;
	> 
	>             include /etc/nginx/default.d/*.conf;
	>   
	>             location / {
	>             }
	> 
	>             error_page 404 /404.html;
	>             location = /40x.html {
	>             }
	> 
	>             error_page 500 502 503 504 /50x.html;
	>             location = /50x.html {
	>             }
	>         }
	> 	    server {
	>             server_name www.pinnuli.com; # managed by Certbot
	>             root        /var/www/pinnuli.github.io;
	>             include /etc/nginx/default.d/*.conf;
	>             location / {
	>        	  }
	>             error_page 404 /404.html;
	>         	  location = /40x.html {
	>             }
	>             error_page 500 502 503 504 /50x.html;
	>             location = /50x.html {
	>             }
	>     	      listen [::]:443 ssl ipv6only=on; # managed by Certbot
	>     	      listen 443 ssl; # managed by Certbot
	>     	      ssl_certificate /etc/letsencrypt/live/www.pinnuli.com/fullchain.pem; # managed by Certbot
	>     	      ssl_certificate_key /etc/letsencrypt/live/www.pinnuli.com/privkey.pem; # managed by Certbot
	>     	      include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
	>     	      ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot
	>     }

- 设置自动更新