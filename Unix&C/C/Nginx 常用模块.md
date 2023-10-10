# Nginx 常用模块

## ngx_http_proxy_module 模块

模块 `ngx_http_proxy_module` 用于将 request 传递到另外一个服务，文档地址：<https://nginx.org/en/docs/http/ngx_http_proxy_module.html> 。

```nginx
location / {
    proxy_pass       http://localhost:8000;
    proxy_set_header Host      $host;
    proxy_set_header X-Real-IP $remote_addr;
}
```

### proxy_pass

指令 proxy_pass 用于设置代理服务的 protocol、address 和一个可选的用于映射 location 的 URI。protocol 可以是 http 和 https，address 可以是域名或者 IP，再加上一个可选的 port。

如果域名可以被解析为多个地址，那么它们将会以 round-robin 的方式被使用。除此之外，address 也可以被指定为一个 [server group](http://nginx.org/en/docs/http/ngx_http_upstream_module.html)。

### proxy_set_header

指令 proxy_set_header 用于重新定义或者追加 request header 发送给代理服务，默认情况下，Nginx 只会重新定义两个 header：

```nginx
proxy_set_header Host       $proxy_host;
proxy_set_header Connection close;
```

## ngx_http_upstream_module 模块

模块 `ngx_http_upstream_module` 用于定义可以被 [proxy_pass](https://nginx.org/en/docs/http/ngx_http_proxy_module.html#proxy_pass)、[fastcgi_pass](https://nginx.org/en/docs/http/ngx_http_fastcgi_module.html#fastcgi_pass)、[uwsgi_pass](https://nginx.org/en/docs/http/ngx_http_uwsgi_module.html#uwsgi_pass)、[scgi_pass](https://nginx.org/en/docs/http/ngx_http_scgi_module.html#scgi_pass)、[memcached_pass](https://nginx.org/en/docs/http/ngx_http_memcached_module.html#memcached_pass) 和 [grpc_pass](https://nginx.org/en/docs/http/ngx_http_grpc_module.html#grpc_pass) 指令引用的服务组，文档地址：<https://nginx.org/en/docs/http/ngx_http_upstream_module.html> 。

```nginx
upstream backend {
    server backend1.example.com       weight=5;
    server backend2.example.com:8080;
    server unix:/tmp/backend3;

    server backup1.example.com:8080   backup;
    server backup2.example.com:8080   backup;
}

server {
    location / {
        proxy_pass http://backend;
    }
}
```

### upstream

指令 upstream 用于定义一组服务。服务可以监听不同的 port。除此之外，服务还可以混合监听 TCP 和 UNIX-domain 两种模式。

```nginx
upstream backend {
    server backend1.example.com weight=5;
    server 127.0.0.1:8080       max_fails=3 fail_timeout=30s;
    server unix:/tmp/backend3;

    server backup1.example.com  backup;
}
```

在默认情况下，Nginx 会采用加权轮训（weighted round-robin）的方式来将请求分配给 upstream 配置的服务。

### server

指令 server 用于定义一个服务的 address 和参数。服务的地址可以是域名或者 IP 再加上一个可选的 port，或者是一个以 `unix:` 为前缀的 UNIX-domain 套接字地址。如果没有指定 port，则默认是 80。如果域名可以被解析为多个 IP 的话，则等同于定义了多个服务。

指令 server 可以定义如下参数：

- weight=number 设置服务的权重，默认是 1；
- max_conns=number 设置可以与代理服务器同时建立的最大连接数，默认是 0，意味着没有限制。
- max_fails=number 设置与代理服务器通讯失败的最大次数。当大于这个阈值时，Nginx 会认为代理服务器已经不可用，并在后续 fail_timeout 时间内避免选择此服务。默认值为 1，设置为 0 表示关闭对此服务的健康检查。
- fail_timeout=time 设置服务被标记为不可用的时长。当经过这个时间之后，Nginx 会开始探测服务是否可用，如果服务可以被正常访问，此服务会被标记为可用。
- ……

### hash

指令 hash 用于指定一个服务组的负载均衡方式：基于 key 的 hash 值在服务之间分配请求。key 可以是文本和变量，或者它们的组合。

### ip_hash

指令 ip_hash 用于指定一个服务组的负载均衡方式：基于客户端 IP 的 hash 值在服务之间分配请求。这种方式可以使同一个客户端的请求始终被映射到同一个服务，除非该服务处于不可用状态。

```nginx
upstream backend {
    ip_hash;

    server backend1.example.com;
    server backend2.example.com;
    server backend3.example.com down;
    server backend4.example.com;
}
```

### least_conn

指令 least_conn 用于指定一个服务组的负载均衡方式：将请求分配给活跃连接数量最少的服务，同时也会考虑服务的权重。

### least_time

指令 least_time 用于指定一个服务组的负载均衡方式：将请求分配给活跃连接数量最少和平均响应时间最短的服务，同时也会考虑服务的权重。

### random

指令 random 用于指定一个服务组的负载均衡方式：将请求分配给随机选择的服务，同时也会考虑服务的权重。

## ngx_http_log_module 模块

模块 `ngx_http_log_module` 用于指定 request 日志的写入格式，文档地址：<https://nginx.org/en/docs/http/ngx_http_log_module.html> 。

```nginx
log_format compression '$remote_addr - $remote_user [$time_local] '
                       '"$request" $status $bytes_sent '
                       '"$http_referer" "$http_user_agent" "$gzip_ratio"';

access_log /spool/logs/nginx-access.log compression buffer=32k;
```

### access_log

指令 access_log 用于指定 request log 的文件地址、写入格式和写入时的缓冲区配置。可以同时配置多个 access_log。以 `syslog:` 为前缀的文件地址表示日志将会被写入 syslog。如果没有配置日志格式，则默认使用 combined 格式。

### log_format

指令 log_format 用于定义 request log 的写入格式。
