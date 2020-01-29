---
title: Nginx
date: 2020-01-09 23:00:06
categories:
tags:
---
# Config Nginx as Load Balancer

The entrance url is `http://192.168.1.108:8080/moodle`, with two servers behind, both open port 80 to accept incoming requests.

Nginx config script lists below:

<!--more-->
```sh
events {
  worker_connections 65535;
}
http {
  log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                    '$status $body_bytes_sent "$http_referer" '
                    '"$http_user_agent" "$http_x_forwarded_for" "$upstream_addr" "$proxy_host" "$http_host" "$host"'
                    ' - "$proxy_add_x_forwarded_for" "$scheme"';

  access_log  logs/access.log  main;
  
  upstream myapp1 {
    ip_hash;
    server 192.168.1.105 weight=8;
    server 192.168.1.108 weight=10;
  }

  server {
    listen 8080;
    location /moodle/ {
      proxy_set_header Host $http_host;
      proxy_set_header  X-Real-IP $remote_addr;
      proxy_set_header  X-Forwarded-For $proxy_add_x_forwarded_for;
      proxy_set_header  X-Forwarded-Proto $scheme;
      proxy_set_header  X-Scheme $scheme;
      proxy_pass http://myapp1;
    }
  }
}
```

The log output is like:
```
192.168.1.107 - - [13/Jan/2020:12:29:31 +1100] "GET /moodle/login/index.php HTTP/1.1" 200 43872 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_2) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/79.0.3945.117 Safari/537.36" "-" "192.168.1.108:80" "myapp1" "192.168.1.108:8080" "192.168.1.108" - "192.168.1.107" "http"
```
We can see the differences between **`"$proxy_host"`, `"$http_host"` and `"$host"`**.

Another thing needs to mention is that location and proxy_pass should be set exactly like above, missing or adding slash, will result in totally different behavior. Set proxy_pass to **`http://myapp1/`**, every requests to `192.168.1.105/moodle`, path is omitted.

## ip_hash
To implement the sticky session, ip_hash should be the silver bullet, as both round robin and least_conn may access different server in a single request.

## Pit of ip_hash

According to the official documentation, **` The first three octets of the client IPv4 address, or the entire IPv6 address, are used as a hashing key`**, which means not entire IPV4 address would be hashed, only the first three numbers are used as a hash key. It is a tricky feature that only explained in official documentation.

To achieve the entire IP address hashed as key, we should use another directive in ngx_http_upstream_module called **`hash`**.

The updated upstream directive should be like: 
``` sh
  upstream myapp1 {
    hash $remote_addr;
    server 192.168.1.105 weight=8;
    server 192.168.1.108 weight=10;
  }
```

ip_hash can cause another problem, if we have a classroom got a lot of students to use our system to attend exam, this will give great burden on our system which can not be solve be ip_hash load balancer.

## Quick explanation of `proxy_set_header`
```
proxy_set_header  X-Real-IP $remote_addr;
proxy_set_header  X-Forwarded-For $proxy_add_x_forwarded_for;
proxy_set_header  X-Forwarded-Proto $scheme;
```
For this part, there are great explanation in official documentation, and used a lot of terminology which hard to understand.

To my understanding, this part is not mandatory to set, but we'd better to have them if you want to track down some issues related to original IP.

This is apache log, with these header set, a original IP will recorded and in server end there might be requests statistic modules are working rely on the original IP. Without this, problems are hard to find and some statistic functions could be malfunctioned.
![apache-log](/blog/img/apache-log.jpg)


# Moodle wwwroot config

```
$CFG->wwwroot   = 'http://192.168.1.108:8080/moodle';
```

# Commands

## Nginx
```
nginx -s reload
# or
systemctl restart nginx

# test configuration file
nginx -c /etc/nginx/nginx.conf -t
# or
service nginx configtest
```


# SSL Configure

```
user  nginx;
worker_processes  1;

error_log  /var/log/nginx/error.log warn;
pid        /var/run/nginx.pid;

events {
    worker_connections  65535;
}

http {
    include       /etc/nginx/mime.types;
    default_type  application/octet-stream;

   	log_format  main  '$remote_addr $remote_user [$time_local] '
                        '$request status:$status $body_bytes_sent '
                        '$http_referer $remote_addr $proxy_add_x_forwarded_for '
                        '$scheme proxy_host:$proxy_host http_host:$http_host host:$host [$http_user_agent] [$upstream_addr] [$request_body]';

    access_log  /var/log/nginx/access.log  main;

    sendfile        on;
    #tcp_nopush     on;

    keepalive_timeout  65;

    #gzip  on;

    #include /etc/nginx/conf.d/*.conf;

    upstream myapp1 {
        #ip_hash;
        #hash $remote_addr;
        server www0.cloudcampus.com.au;
        server www5.cloudcampus.com.au;
        server www3.cloudcampus.com.au:8080;
    }

    server {
        listen 443 ssl;
        #listen 8080;
        server_name www.cloudcampus.com.au;
        ssl_certificate        /etc/ssl/certs/cloudcampus_com_au2019.crt;
        ssl_certificate_key    /etc/ssl/certs/private2019.key;
        ssl_client_certificate /etc/ssl/certs/cloudcampus_com_au-ca2019.crt;
        #ssl_verify_client    A  optional;
        ssl_session_cache shared:SSL:20m;
        ssl_session_timeout 10m;

        location / {
            proxy_pass http://myapp1;
            proxy_set_header Host $http_host;
            proxy_set_header  X-Real-IP $remote_addr;
            proxy_set_header  X-Forwarded-For $proxy_add_x_forwarded_for;
            proxy_set_header  X-Forwarded-Proto $scheme;
            proxy_set_header  X-Scheme $scheme;
            proxy_buffering        on;
        }
    }
}
```