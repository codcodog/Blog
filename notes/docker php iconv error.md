docker php iconv error
======================

### 场景

PHP docker 镜像，调用 `iconv` 函数报错：`iconv(): Wrong charset, conversion from 'utf-8' to 'GBK' is not allowed`

基础镜像是：
- Alpine Linux 3.4.6
- PHP 5.6.29


### 方案

`Alpine Linux` 自带的 `libiconv` 库版本太低导致.

在原有的 `Dockerfile` 新增两条指令：
```
RUN apk add --no-cache --repository http://dl-cdn.alpinelinux.org/alpine/v3.12/community/ gnu-libiconv=1.15-r2
ENV LD_PRELOAD /usr/lib/preloadable_libiconv.so
```

### 参考
[Proper iconv](https://github.com/docker-library/php/issues/240)
