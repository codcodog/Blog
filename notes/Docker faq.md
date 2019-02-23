Docker FAQ
===========

1. Dockerfile 中 `COPY` 指令和 `ADD` 指令有什么区别  

    简单来说，最主要的区别是 `ADD` 功能更为强大一些:
    - `ADD` 允许 `<src>` 是一个 URL
    - 如果 `ADD` 的 `<src>` 参数是可以识别的压缩格式存档文件，则对其进行解压
    > 如果 ADD 带一个 URL 是 xxx.tar.gz 的文件的话，是不会对其进行解压的

    「书写 Dockerfiles 的最佳实践」推荐使用 `COPY` 指令而不是使用 `ADD` 指令.

2. Dockerfile 中 `CMD` 指令和 `ENTRYPOINT` 指令有什么区别  

    Docker 有一个默认的「入口点」 `/bin/sh -c`，但没有一个默认的命令.
    > 这里论述，感觉不对，主要看 Dockerfile 中的 `CMD` 和 `ENTRYPOINT` 是基于 exec 还是 shell 模式  
    > 若是 shell 模式则是利用 /bin/sh -c 调起命令执行的，exec 模式则是直接执行命令

    当你这样运行 `docker` 的时候：`docker run -i -t ubuntu bash`，「入口点」就是默认的 `/bin/sh -c`，镜像是 `ubuntu`，命令是
    `bash`.  
    该命令是通过「入口点」运行的，实际上执行的是 `/bin/sh -c bash`. 这允许 Docker 依靠 `shell` 的解析器快速实现 `RUN`.  
    后来，用户要求能够自定义它，因此引入了 `ENTRYPOINT` 和 `--entrypoint`.

    在上面的示例中，`ubuntu` 之后的内容都是命令并且传递给了「入口点」.  
    当使用 `CMD` 指令的时候，实际上就跟 `docker run -i -t ubuntu <cmd>` 一样，`<cmd>` 就是「入口点」的参数.  
    你仍然会在容器中启动一个 bash shell 是因为 `ubuntu` 的 `Dockerfile` 指定了一个默认的 `CMD`: `CMD ["bash"]`

    因为所有内容都会传递到「入口点」，所以你可以从「镜像」获取到非常好的行为.  
    例如，把「镜像」用作「二进制」命令.  
    当把 `["/bin/cat"]` 作为「入口点」并执行 `docker run img /etc/passwd` 的时候，你将会得到，
    由于 `/etc/passwd` 是命令并传递给「入口点」，因此最终执行的只是 `/bin/cat /etc/passwd`.

    另外一个例子是将任何 `cli` 作为「入口点」.  
    假设有一个 `redis` 镜像，不用运行 `docker run redisimg redis -H something -u toto get key`，而是简单定义
    `ENTRYPOINT ["reids", "-H", "something", "-u", "toto"]`，然后运行类似这样的命令将会得到相同的结果：
    `docker run redisimg get key`.

3. 从主机复制文件到 Docker 容器

    `cp` 命令能够用来复制文件. 可以复制一个特定的文件，如
    ```
    docker cp foo.txt mycontainer:/foo.txt
    docker cp mycontainer:/foo.txt foo.txt
    ```
    特别强调，`mycontainer` 是「容器」ID 而不是「镜像」ID.

    文件夹 `src` 包含多个文件可以使用以下方法复制到 `target` 文件夹中
    ```
    docker cp src/. mycontainer:/target
    docker cp mycontainer:/src/. target
    ```
    参考：[Docker CLI docs for cp](https://docs.docker.com/engine/reference/commandline/cp/)

    小于 Docker 1.8 的版本，只能将文件从容器复制到主机，而不能从主机复制到容器.

4. 如何移除旧的 Docker 容器

    使用 `docker container prune`.

    也可以使用 `docker system prune`, 将会在一个命令中移除「容器」(containers)，「镜像」(images)，「卷」(volumes)以及「网络」(networks).

5. 如何从主机获取 Docker 容器的 IP 地址

    `inspect` 的 `--format` 选项可以获取到.  
    现代 Docker 客户端语法
    ```
    docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' container_name_or_id
    ```
    古代 Docker 客户端语法
    ```
    docker inspect --format '{{ .NetworkSettings.IPAddress }}' container_name_or_id
    ```

    > 若是 windows 系统，则需要双引号(“)而不是单引号(‘)

6. 在不使用仓库的情况下，如何将 Docker 镜像从一台主机复制到另一台

    首先，把镜像保存为一个 `tar` 文件.
    ```
    docker save -o < path for generated tar file > < image name >
    ```
    然后使用常规文件传输工具（如 `cp` 或 `scp`）将镜像复制到系统.   
    之后再将镜像导入到 Docker 中.
    ```
    docker load -i < path to image tar file >
    ```

7. 如何处理 Docker 中的持久存储（例如：数据库）

    使用 [volume API](https://docs.docker.com/engine/reference/commandline/volume_create/)
    ```
    docker volume create --name hello
    docker run -d -v hello:/container/path/for/volume container_image my_command
    ```
    这意味着必须放弃仅数据容器模式以支持新卷.

    实际上 `volume API` 只是实现数据容器模式的更好的方法.

    如果使用 `-v volume_name:/container/fs/path` 创建容器，Docker 将自动为你创建一个命名卷，它可以  
    1. 通过 `docker volume ls` 列出来
    2. 通过 `docker volume inspect volume_name` 识别
    3. 备份为普通目录
    4. 像以前一样通过 `--volumes-from` 连接备份

    新的 `volume API` 增加了一个有用的命令，能够让你识别出悬空的卷
    ```
    docker volume ls -f dangling=true
    ```
    然后可以通过卷名移除
    ```
    docker volume rm < volume name >
    ```

    可以只通过一条命令移除所有悬空的卷
    ```
    docker volume rm $(docker volume ls -f dangling=true -q)

    # Or using 1.13.x
    docker volume prune
    ```

8. 如何在 Docker 中列出容器

    只显示正在运行的「容器」
    ```
    docker ps
    ```

    显示所有的「容器」
    ```
    docker ps -a
    ```

    显示最新创建的「容器」（包括所有状态）
    ```
    docker ps -l
    ```

    显示 N 个最新创建的「容器」（包括所有状态）
    ```
    docker ps -n=-1
    ```

    显示总的文件大小
    ```
    docker ps -s
    ```

    在新版本的 Docker, 命令有更新，新增了一些管理命令  

    列出正在运行的「容器」
    ```
    docker container ls
    ```

    列出所有的容器（不管状态）
    ```
    docker container ls -a
    ```

9. 如何删除 Docker 中的镜像

    使用 `docker rmi node`

    列出所有创建的「容器」 `docker ps -a`

    移除所有现有的「容器」（不是「镜像」），执行 `docker rm $(docker ps -aq)`

10. 如何进入 Docker 容器的 shell

    使用 `docker exec`, 例如
    ```
    docker exec -it < mycontainer > bash
    ```

11. Docker 镜像存储在主机那个位置

    `/var/lib/docker` 目录的内容主要依赖于 [driver Docker is using for storage](https://github.com/moby/moby/blob/990a3e30fa66e7bd3df3c78c873c97c5b1310486/daemon/graphdriver/driver.go#L37-L43)

    默认是 `aufs`，但也可以为 `overlay`, `overlay2`, `btrfs`, `devicemapper` 或者 `zfs`，主要是取决于你的「内核」支持.  
    大多数情况下都会是 `aufs`，但是 `RedHats` 的是 `devicemapper`.

    可以使用 `-s` 或者 `--storage-dirver=` 选项为 `Docker daemon` 手动设置存储的驱动.
    - `/var/lib/docker/{dirver-name}` 包含了指定存储驱动的镜像内容
    - `/var/lib/docker/graph/{id}` 仅包含镜像的元数据，存储在 `json` 和 `layersize` 文件

    若是 `aufs`
    - `/var/lib/docker/aufs/diff/{id}` 包含了镜像文件内容
    - `/var/lib/docker/repositories-aufs` 是一个 `JSON` 文件，包含了本地镜像信息. 可以使用 `docker images` 命令查看

    若是 `devicemapper`
    - `/var/lib/docker/devicemapper/devicemapper/data` 存储了镜像
    - `/var/lib/docker/devicemapper/devicemapper/metadata` 存储了元数据
    > 这些文件是精简配置的’稀疏‘文件，因此不像它们看起来那么大

12. Docker 容器和镜像的区别

    「镜像」的运行实例被称作「容器」.

    一个「镜像」，实质是一组 `layer` 层的描述. 当你启动此「镜像」，则会有一个运行此「镜像」的「容器」. 可以创建许多同一个「镜像」的运行「容器」.

    `docker images` 可以查看所有的「镜像」, 而 `docker ps` 可以查看所有运行中的「容器」, `docker ps -a` 查看所有的「容器」.

    因此，「镜像」的运行实例就是「容器」.

13. 如何移除旧的不用的 Docker 镜像

    `docker system prune` 会删除所有的悬空数据（例如：停止运行的「容器」，没有「容器」的「卷」，没有「容器」的「镜像」），甚至不使用的数据，使用 `-a` 选项.

    还可以使用：
    - docker container prune
    - docker image prune
    - docker network prune
    - docker volume prune

    对于不使用的「镜像」，使用 `docker image prune -a` （删除悬空和不使用的「镜像」）.
    > 不使用的「镜像」是指未被任何「容器」引用

    把 `--filter` 选项与 `docker xxx prune` 组合使用，是一种限制 `prune` 的好方法.

14. 如何将环境变量传递给 Docker 容器

    使用 `-e` 选项.
    ```
    sudo docker run -d -t -i -e REDIS_NAMESPACE='staging' \
    -e POSTGRES_ENV_POSTGRES_PASSWORD='foo' \
    -e POSTGRES_ENV_POSTGRES_USER='bar' \
    -e POSTGRES_ENV_DB_NAME='mysite_staging' \
    -e POSTGRES_PORT_5432_TCP_ADDR='docker-db-1.hidden.us-east-1.rds.amazonaws.com' \
    -e SITE_URL='staging.mysite.com' \
    -p 80:80 \
    --link redis:redis \
    --name container_name dockerhub_id/image_name
    ```

    或者，不想在 `ps` 命令行看到这样赋值的参数，则可以这样
    ```
    sudo PASSWORD='foo' docker run  [...] -e PASSWORD [...]
    ```

    如果有很多环境变量特别是它们出于安全考虑不能暴露的话，可以使用 `env-file`.
    ```
    $ docker run --env-file ./env.list ubuntu bash
    ```
    > `env.list` 文件每一行要符合 `VAR=VAL` 这样的格式

15. 强制 Docker 不使用缓存构建镜像

    使用 `--no-cache` 选项.
    ```
    docker build --no-cache -t u12_core .
    ```

16. Docker 如何修改仓库名或者镜像名

    `docker tag server:latest myname/server:latest`  

    或者

    `docker tag d583c3ac45fd myname/server:latest`

    如果不要旧的名字，在重新标记之后可以移除它
    ```
    docker rmi server
    ```
    这仅仅移除 `alias/tag`. 因为 `d583c3ac45fd` 有其他名字，真实的「镜像」不会被移除.

17. 在 Docker 中 expose 和 publish 有什么不同

    `expose` 是一种文档化的方式，与 `Dockerfile` 相关.
    > expose 只是定义了那些端口被使用，是一种文档化定义，实际上不进行端口映射或打开端口的操作

    `--publish`(`-p`) 是一种将主机端口映射到正在运行的容器端口的方法，与 `docker run` 相关.
    > -p 会告诉容器在网络接口打开那些端口

18. 如何 attach 和 detach Docker 进程

    使用 `CTRL-c` `detach` 「容器」，会导致 Docker 「容器」退出.

    使用 `--sig-proxy` 选项，可以避免「容器」退出.
    ```
    docker attach --sig-proxy=false 304f5db405ec
    ```

    使用 `CTRL-p CTRL-q` 也可以 `detach` 而「容器」不退出, 不过只有在「容器」是使用 `-t` 和 `-i` 选项启动的时候，才会生效.
    ```
    $ docker run -i -t -d nginx
    af203c3e997b94c7a519463c26c9290c777942556be4682738a8892af1f7062b

    $ docker ps
    CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
    af203c3e997b        nginx               "nginx -g 'daemon of…"   3 seconds ago       Up 2 seconds        80/tcp              keen_lumiere

    $ docker attach keen_lumiere
    read escape sequence # 这里使用 CTRL-p CTRL-q detach 「容器」

    $ docker ps # 「容器」没有退出
    CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS              PORTS               NAMES
    af203c3e997b        nginx               "nginx -g 'daemon of…"   20 seconds ago      Up 19 seconds       80/tcp              keen_lumiere
    ```

    也可以自定义 `detach` 键：`--detach-keys`  
    详情参考：[set your own detach sequence](https://docs.docker.com/engine/reference/commandline/attach/)

19. Docker daemon 日志的位置

    这主要取决于你的系统，以下是几个位置以及包含几个操作系统的命令.
    - Ubuntu(使用 `upstart`) - `/var/log/upstart/docker.log`
    - Ubuntu(使用 `systemd`) - `journalctl -fu docker.service`
    - Amazon Linux AMI - `/var/log/docker`
    - Boot2Docker - `/var/log/docker.log`
    - Debian GNU/Linux - `/var/log/daemon.log`
    - CentOS - `/var/log/daemon.log | grep docker`
    - CoreOS - `journalctl -u docker.service`
    - Fedora - `journalctl -u docker.service`
    - Red Hat Enterprise Linux Server - `/var/log/messages | grep docker`
    - OpenSuSE - `journalctl -u docker.service`

20. Dockerfile 中如何更新 PATH 环境变量

    使用 [Environment replacement](https://docs.docker.com/engine/reference/builder/#environment-replacement)
    ```
    ENV PATH="/opt/gtk/bin:${PATH}"
    ```

21. 如何检查 `docker build` 失败的文件系统

    `Docker` 每次从 `Dockerfile` 成功执行一个 `RUN` 指令，都会提交一个新的 `layer` 层到「镜像」文件系统.  
    你可以方便地使用这些 `layer` 层 ID 作为一个「镜像」启动一个新的「容器」.

    使用这样的 `Dockerfile`
    ```
    FROM busybox
    RUN echo 'foo' > /tmp/foo.txt
    RUN echo 'bar' >> /tmp/foo.txt
    ```
    构建一个「镜像」
    ```
    $ docker build -t so-2622957 .
    Sending build context to Docker daemon 47.62 kB
    Step 1/3 : FROM busybox
    ---> 00f017a8c2a6
    Step 2/3 : RUN echo 'foo' > /tmp/foo.txt
    ---> Running in 4dbd01ebf27f
    ---> 044e1532c690
    Removing intermediate container 4dbd01ebf27f
    Step 3/3 : RUN echo 'bar' >> /tmp/foo.txt
    ---> Running in 74d81cb9d2b1
    ---> 5bd8172529c1
    Removing intermediate container 74d81cb9d2b1
    Successfully built 5bd8172529c1
    ```

    你可以使用 `00f017a8c2a6`, `044e1532c690` 和 `5bd8172529c1` 启动一个新的「容器」.
    ```
    $ docker run --rm 00f017a8c2a6 cat /tmp/foo.txt
    cat: /tmp/foo.txt: No such file or directory

    $ docker run --rm 044e1532c690 cat /tmp/foo.txt
    foo

    $ docker run --rm 5bd8172529c1 cat /tmp/foo.txt
    foo
    bar
    ```

    当 `Dockerfile` 中的一个指令执行失败时，你需要做的是查找上一个 `layer` 层的 ID, 并在从该 ID 创建的「容器」中运行 `shell`
    ```
    docker run --rm -it <id_last_working_layer> bash -il
    ```
    一旦进入到「容器」
    - 尝试失败的命令，重现该问题
    - 然后修复该命令并测试它
    - 最后使用修复之后的命令更新你的 `Dockerfile`

    若是想直接在失败的 `layer` 层排查问题而不是在最后一个工作的 `layer` 层，则可以：  
    首先，找到失败的「容器」
    ```
    $ docker ps -a
    CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                          PORTS               NAMES
    6934ada98de6        42e0228751b3        "/bin/sh -c './utils/"   24 minutes ago      Exited (1) About a minute ago                       sleepy_bell
    ```

    然后，把它提交到一个「镜像」
    ```
    $ docker commit 6934ada98de6
    sha256:7015687976a478e0e94b60fa496d319cdf4ec847bcd612aecf869a72336e6b83
    ```

    之后，运行这个「镜像」
    ```
    $ docker run -it 7015687976a4 [bash -il]
    ```
    这样，就处于构建失败时的状态了.
