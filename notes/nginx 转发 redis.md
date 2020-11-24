nginx 转发 redis
==================

### 场景

本地开发，需要连接测试环境的 `redis`，但 `redis` 是内网的，不对外网开放.


### 方案

利用 `nginx` 进行反向代理，把请求转发到内网的 `redis` 上.  
找到测试环境的一台 `nginx`，配置反向代理.

修改 `nginx.conf`
```
http {
    ...
}

# 注意跟 http 同级
stream {
     upstream redis {
         server  10.8.xx.xxx:6379 max_fails=3 fail_timeout=30s;
     }

     server {
          listen 16379;
          proxy_connect_timeout 1s;
          proxy_timeout 3s;
          proxy_pass redis;
     }
}
```

如是不想修改 `nginx.conf`，而是在 `conf.d` 添加新的配置文件 `redis.conf`，则需要把 `include` 指令从 `http` 移出来  
> 这样保证添加的 `redis.conf` 跟 `http` 同级
```
http {
    ...
}

include /etc/nginx/conf.d/*.conf
```

若遇到 `nginx: [emerg] unknown directive "stream" in /usr/local/openresty/nginx/conf/conf.d/redis.conf:1` 错误  
是因为 `stream` 模块没加载到，手动加载下
```
load_module /usr/lib64/nginx/modules/ngx_stream_module.so;

http {
    ...
}
```

如果不知道 `ngx_stream_module.so` 的路径在那里，则全局搜一下
```
$ find / -name 'ngx_stream_module'
```

最后，重新加载 `nginx` 下配置文件
```bash
$ nginx -s reload

# 查看代理端口是否成功
$ ss -atn | grep 16379
```
