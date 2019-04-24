title: SSL生成
author: tr
tags:
  - ssl
categories:
  - ssl
date: 2019-04-04 09:06:00
---
### ssl本地生成脚本

在公司部署前端到nginx服务器，遇到了证书问题，gateway有一个证书，前端有一个证书。

<!--more-->

#### 本地生成脚本

```shell

#!/bin/sh

# create self-signed server certificate:

read -p "Enter your domain [www.example.com]: " DOMAIN

echo "Create server key..."

openssl genrsa -des3 -out $DOMAIN.key 1024

echo "Create server certificate signing request..."

SUBJECT="/C=US/ST=Mars/L=iTranswarp/O=iTranswarp/OU=iTranswarp/CN=$DOMAIN"

openssl req -new -subj $SUBJECT -key $DOMAIN.key -out $DOMAIN.csr

echo "Remove password..."

mv $DOMAIN.key $DOMAIN.origin.key
openssl rsa -in $DOMAIN.origin.key -out $DOMAIN.key

echo "Sign SSL certificate..."

openssl x509 -req -days 3650 -in $DOMAIN.csr -signkey $DOMAIN.key -out $DOMAIN.crt

echo "TODO:"
echo "Copy $DOMAIN.crt to /etc/nginx/ssl/$DOMAIN.crt"
echo "Copy $DOMAIN.key to /etc/nginx/ssl/$DOMAIN.key"
echo "Add configuration in nginx:"
echo "server {"
echo "    ..."
echo "    listen 443 ssl;"
echo "    ssl_certificate     /etc/nginx/ssl/$DOMAIN.crt;"
echo "    ssl_certificate_key /etc/nginx/ssl/$DOMAIN.key;"
echo "}"

```
这是廖雪峰老师的脚本，很方便直接用，再写一个脚本导入到前端项目。

[https://www.liaoxuefeng.com/article/0014189023237367e8d42829de24b6eaf893ca47df4fb5e000]

#### 问题

在部署的时候遇到了一个很奇怪的问题，我直接访问项目入口会被提示证书认证错误，去后台gateway查看的时候发现日志提示证书unknown。再访问一遍gateway即可。

思路：应该是访问前端的时候获得了前端的证书，用证书访问会走一次网关，但是网关自己也有证书，不认前端的证书，这时候访问网关，获得网关的证书，再次访问即可识别。