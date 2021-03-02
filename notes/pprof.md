pprof
======


### 基本使用

查看堆栈调用信息
```bash
$ go tool pprof http://localhost:6060/debug/pprof/heap
```

查看协程信息
```bash
$ go tool pprof http://localhost:6060/debug/pprof/goroutine
```

查看30s内的 CPU 信息
```bash
$ go tool pprof http://localhost:6060/debug/pprof/profile?seconds=30
```

其他
```bash
$ go tool pprof http://localhost:6060/debug/pprof/threadcreate

$ go tool pprof http://localhost:6060/debug/pprof/block

$ go tool pprof http://localhost:6060/debug/pprof/mutex

$ go tool pprof http://localhost:6060/debug/pprof/trace?seconds=5
```

### 扩展使用
排查 `golang` 程序内存泄露.

获取 A 时间点的堆栈 profile
```bash
$ curl -s http://localhost:6060/debug/pprof/heap > A.heap
```

过一段时间后，内存开始泄露了，再获取 B 时间点的堆栈 profile
```bash
$ curl -s http://localhost:6060/debug/pprof/heap > B.heap
```

比较 A, B 时间点的堆栈差异
```bash
$ go tool pprof --base A.heap B.heap
```

使用 `web` 命令生成一个 SVG 文件
```bash
(pprof) web
```

或者直接打开 web 界面
```
$ go tool pprof --http :9090 --base B.heap A.heap
```

### 参考
[Profiling Go programs with pprof](https://jvns.ca/blog/2017/09/24/profiling-go-with-pprof/)  
[Hi, 使用多年的go pprof检查内存泄漏的方法居然是错的](https://colobu.com/2019/08/20/use-pprof-to-compare-go-memory-usage/)  
