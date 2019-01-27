Jenkins PHP 部署示例
=====================

### 搭建 Jenkins
在本地搭建 `Jenkins`.  

> 参考 [jenkins 安装](https://github.com/codcodog/Blog/issues/106)

### 配置 SSH
配置 `ssh`, 程序在 `Jenkins` 服务器打包完之后，发布到目标服务器上.

`Manage Jenkins` -> `Configure System` -> `Publish over SSH` -> `Add`

![selection_011](https://user-images.githubusercontent.com/18098145/51800736-b03c4e80-226e-11e9-8550-3c04fb02ac22.png)

这里需要特别注意的是 `Remote Directory`.  
> 此目录必须存在，不会自动创建. 而且发布机（Jenkins）只能将文件放在此目录下.

### 配置项目

- 创建一个项目

![selection_012](https://user-images.githubusercontent.com/18098145/51801044-89cce200-2273-11e9-994f-6e05ef20b6a0.png)

- 输入项目名字，选择 `Freestyle project` 

![selection_013](https://user-images.githubusercontent.com/18098145/51800912-cdbee780-2271-11e9-91ce-b9959763223c.png)

- 添加项目仓库

![selection_014](https://user-images.githubusercontent.com/18098145/51801000-0ca16d00-2273-11e9-88e9-360568d7ac60.png)

- 配置构建

![selection_015](https://user-images.githubusercontent.com/18098145/51801066-e7f9c500-2273-11e9-8a44-3d6995982b3d.png)

![selection_016](https://user-images.githubusercontent.com/18098145/51801093-3313d800-2274-11e9-9e71-3f1612751c71.png)

```
project=blog
desc_name=/tmp/$project.$BUILD_ID.$BUILD_NUMBER.tar.gz

composer install --no-dev

tar -cvz  --exclude='.git' --exclude=$desc_name -f $desc_name .
mv $desc_name .
```
这里使用 `.` 而不是 `./*`, `./*` 不会压缩隐藏文件  
项目里会包含配置文件 `.env.example` 这种隐藏文件.

- 构建完之后，发布到目标服务器

![selection_017](https://user-images.githubusercontent.com/18098145/51801166-2ba0fe80-2275-11e9-8389-9cf85f6521a9.png)

![selection_018](https://user-images.githubusercontent.com/18098145/51801289-c1895900-2276-11e9-943e-918b47e9c487.png)

```
PREFIX=/data/www
LIB=$PREFIX/lib

TIME=$(date +%Y%m%d%H%M%S)

DESC_NAME=blog."$TIME"
DESC=$LIB/$DESC_NAME

LINK_NAME=blog
LINK_DESC=$PREFIX/$LINK_NAME

mkdir -p $DESC
tar -xzf /tmp/demo/blog.$BUILD_ID.$BUILD_NUMBER.tar.gz -C $DESC
cd $DESC
cp .env.example .env

ln -snf $DESC $LINK_DESC
```

配置完成，保存，点击 `Build Now` 就可以了.
