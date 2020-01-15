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
    #location ~ .*\.(html|htm|png|jpg|gif|jpeg|bmp|ico|txt|js|css)$ {
    #       proxy_pass http://myapp1;
    #}
    #location  ~ ^/(.*)$ {
    #        proxy_pass   http://myapp1/moodle/$1;
    #}
  }
}
```

The log output is like:
```
192.168.1.107 - - [13/Jan/2020:12:29:31 +1100] "GET /moodle/login/index.php HTTP/1.1" 200 43872 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_2) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/79.0.3945.117 Safari/537.36" "-" "192.168.1.108:80" "myapp1" "192.168.1.108:8080" "192.168.1.108" - "192.168.1.107" "http"
```
We can see the differences between **`"$proxy_host"`, `"$http_host"` and `"$host"`**.

## ip_hash
To implement the sticky session, ip_hash should be the silver bullet, as both round robin and least_conn may access different server in a single request.

## Pit of ip_hash

According to the official documentation, **` The first three octets of the client IPv4 address, or the entire IPv6 address, are used as a hashing key`**, which means not entire IPV4 address would be hashed, only the first three numbers are used as a hash key. It costs a great effort to find this tricky feature.

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
```