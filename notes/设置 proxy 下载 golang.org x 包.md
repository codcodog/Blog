设置 proxy 下载 golang.org/x/
==============================

`Go 1.13` 以上.

设置 `Go ENV`
```bash
$ go env -w GO111MODULE=on
$ go env -w GOPROXY=https://goproxy.cn,direct
```

#### 参考
[goproxy.cn](https://github.com/goproxy/goproxy.cn)
