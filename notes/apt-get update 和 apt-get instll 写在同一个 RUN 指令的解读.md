apt-get update 和 apt-get install 写在同一个 RUN 指令的解读
===========================================================

### 场景
看到很多 `Dockerfile` 都是把 `apt-get update` 和 `apt-get install` 写在同一个 `RUN` 指令中的
```
RUN apt-get update && apt-get install -y \
        package-bar \
        package-baz \
        package-foo
```

而不是
```
RUN apt-get update
RUN apt-get install -y \
        package-bar \
        package-baz \
        package-foo
```

只知道这样写在同一个 `RUN` 中会减少 `layer` 层，缩减构建镜像的大小.  

但看到有一些文章提到，只有写在同一个 `RUN` 中才会对后面的 `apt-get install` 生效，却没有细说原因.  

因此产生了一个疑惑，分开写也应该会对后面的 `apt-get install` 生效才对啊，因为镜像的构建是一层一层的，后面的层会基于前面的层.  
也就是说，`RUN apt-get update` 会单独构建一层，并且会对后面 `RUN apt-get install` 的层产生作用才对.

### 分析
假设有一个这样的 `Dockerfile`
```
FROM ubuntu:14.04
RUN apt-get update
RUN apt-get install -y curl
```

构建镜像之后，所有的层都会在 Docker 的缓存中.  
假设后来修改 `apt-get install` 添加额外的包
```
 FROM ubuntu:14.04
 RUN apt-get update
 RUN apt-get install -y curl nginx
```

Docker 将初始和修改的指令视为相同，并会重用之前构建的缓存.  
因此，不会执行 `apt-get update`，因为构建直接使用之前的缓存版本.  
由于 `apt-get update` 没有运行，所以构建安装的 `curl` 和 `nginx` 包很可能是过时的版本.

所以，把 `apt-get update` 和 `apt-get install` 写在同一个 `RUN` 中以获取最新版本的包，而且还减少了 `layer` 层.

**参考**  
[Best practices for writing Dockerfiles](https://docs.docker.com/develop/develop-images/dockerfile_best-practices/#run)

