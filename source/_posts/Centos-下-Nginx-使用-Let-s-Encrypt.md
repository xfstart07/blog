---
title: Centos 下 Nginx 使用 Let's Encrypt
date: 2018-10-11 16:45:52
categories: Devops
tags:
    - SSL
    - Nginx
    - Centos
---

Certbot 的[官方网站](https://certbot.eff.org)，系统环境是 Centos 7，Nginx 1.12。

### 获取 Certbot 客户端


```
wget https://dl.eff.org/certbot-auto
chmod a+x ./certbot-auto
./certbot-auto --help
```

### 配置 nginx 、验证域名所有权

在配置文件（ /etc/nginx/conf.d/blog.ssl.conf ）中添加如下内容，这一步是为了通过 Let’s Encrypt 的验证，验证 blog.weixinote.com 这个域名是属于我的管理之下。

```
# certbot
location ~ /.well-known {
		allow all;
}
```

修改配置后重启`nginx -s reload`。

<!-- more -->

### 生成证书

```
./certbot-auto certonly --webroot -w /data/website/weixinote.com -d  blog.weixinote.com
```

中间会有一些自动运行及安装的软件，不用管，让其自动运行就好，有一步要求输入邮箱地址的提示，照着输入自己的邮箱即可，顺利完成的话，屏幕上会有提示信息。

### 配置 Nginx，使用 SSL 证书

```
server {
	listen 443;
	server_name blog.weixinote.com;
	charset utf-8;

	client_max_body_size 50M;
    ssl on;
    ssl_certificate /etc/letsencrypt/live/blog.weixinote.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/blog.weixinote.com/privkey.pem;
    ssl_session_timeout 5m;
    ssl_ciphers ECDHE-RSA-AES128-GCM-SHA256:ECDHE:ECDH:AES:HIGH:!NULL:!aNULL:!MD5:!ADH:!RC4;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_prefer_server_ciphers on;

	root /data/website/weixinote.com;
	index index.html index.htm;

	access_log  /data/logs/nginx/blog.log;

	# certbot
	location ~ /.well-known {
    		allow all;
	}
}
```

然后我们在加上将 http 全部重定向到 https 上

```
server {
    listen       80;
    server_name  blog.weixinote.com;
    rewrite ^(.*) https://$host$1 permanent;
}
```

然后重启 `nginx -s reload` 。

### 使用 acme.sh 启动更新

安装 acme.sh

```
curl  https://get.acme.sh | sh
```

[acme.sh文档](https://github.com/Neilpang/acme.sh/wiki/%E8%AF%B4%E6%98%8E)

## 资源

* https://linuxstory.org/deploy-lets-encrypt-ssl-certificate-with-certbot/
* https://github.com/Neilpang/acme.sh
* https://ruby-china.org/topics/31942
* https://certbot.eff.org
* https://imququ.com/post/troubleshooting-https.html