vim-go no ParseGoHandle for file
=================================

### 场景

项目使用 `govendor` 管理的，并且不在 `GOPATH` 下，使用 `vim-go` 的 `gd` 报错：`vim-go: no ParseGoHandle for file`.

### 原因

主要跟 `go` 的版本有关.

系统 `go` 的版本为
```bash
$ go version 
go version go1.13.4 linux/amd64
```

#### 如何解决

直接使用低版本 `go`，这里使用 `go 1.11`.
> 项目是使用 go 1.11 构建的，所以跟项目保持一致.

之后，直接 `export GOPATH=项目路径`，再使用 `vim` 打开即可.
```bash
$ echo $GOPATH
/home/cryven/.go

$ cd ~/coding/product_query/
$ export GOPATH=`pwd`
$ vim
```

若是使用 `go 1.13` 的话，则需要设置 `GOFLAGS=-mod=vendor`，`GO111MODULE=off`
```bash
$ cd ~/coding/product_query/
$ export GOFLAGS=-mod=vendor
$ export GO111MODULE=off
$ export GOPATH=`pwd`
$ vim
```

### 参考
[no ParseGoHandle for file](https://github.com/fatih/vim-go/issues/2628)
