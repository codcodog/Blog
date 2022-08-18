构建镜像设置 go proxy
======================

### 场景

在构建 Golang 项目镜像的时候，由于国内网络的原因，需要配置 `Goproxy` 才行，要不某些包没法拉取.


### 如何配置

直接在 `Dockerfile` 添加
```
ENV GOPROXY=https://goproxy.cn,direct
```

`Goproxy` 必须添加在下载包指令之前，要不没法生效.
```
FROM golang:1.18 as builder

WORKDIR /workspace
COPY go.mod go.mod
COPY go.sum go.sum

# 注意，这里需要在 go mod download 之前添加
ENV GOPROXY=https://goproxy.cn,direct

RUN go mod download
...
```
