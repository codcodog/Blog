jenkins 安装
=============

#### 安装

##### 下载 `jenkins`
```
$ mkdir /usr/local/jenkins
$ cd /usr/local/jenkins
$ wget -c http://mirrors.jenkins.io/war-stable/latest/jenkins.war
```
下载地址: [jenkins](https://jenkins.io/doc/pipeline/tour/getting-started/)

##### 配置环境变量
设置 `jenkins` 的主目录
```
## root 用户配置 /etc/profile
# vi /etc/profile
export JENKINS_ROOT=/usr/local/jenkins
export JENKINS_HOME=$JENKINS_ROOT/jenkins_home
# source /etc/profile

## 非 root 用户，配置 .bashrc
$ vi ~/.bashrc
export JENKINS_ROOT=/usr/local/jenkins
export JENKINS_HOME=$JENKINS_ROOT/jenkins_home
$ source ~/.bashrc
```

##### 启动
```
$ nohup java -jar /usr/local/jenkins/jenkins.war --httpPort=8080 >> /usr/local/jenkins/jenkins.log 2>&1 &
```

##### 配置
`web` 配置
```
http://localhost:8080/
```

#### 问题
根据 `web` 配置完之后，登录进来空白页.
> 重启 jenkins 即可.

#### 其他
`jenkins` 重启
```
http://localhost:8080/restart
```

`jenkins` 重载 - 重新加载配置信息
```
http://localhost:8080/reload
```

`jenkins` 退出
```
http://localhost:8080/exit
```


#### 参考文章  
[Jenkins安装](https://www.jianshu.com/p/195fc1588df8)
