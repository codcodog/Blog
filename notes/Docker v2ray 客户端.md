Docker v2ray 客户端
====================

### 场景

`Arch Linux` 环境使用 `Docker` 搭建 `v2ray` 客户端


### 方案

先拉取 `v2ray` 镜像
```bash
$ docker pull v2fly/v2fly-core
```

配置文件 `config.json`
```
{
    "inbounds": [
        {
            "port": 1080,
            "listen": "127.0.0.1",
            "protocol": "socks",
            "settings": {
                "udp": true
            }
        }
    ],
    "outbounds": [
        {
            "protocol": "vmess",
            "settings": {
                "vnext": [
                    {
                        "address": "server", // 服务器地址
                        "port": 30982, // 端口号
                        "users": [
                            {
                                "id": "id", // 用户ID
                                "alterId": 64
                            }
                        ]
                    }
                ]
            }
        },
        {
            "protocol": "freedom",
            "tag": "direct"
        }
    ],
    "routing": {
        "domainStrategy": "IPOnDemand",
        "rules": [
            {
                "type": "field",
                "ip": [
                    "geoip:private"
                ],
                "outboundTag": "direct"
            }
        ]
    }
}
```

运行镜像
```bash
# 这里直接使用主机网络
$ docker run -d --name v2ray --network host -v /home/cryven/v2ray/config.json:/etc/v2ray/config.json v2fly/v2fly-core
```

配置浏览器 `SwitchyOmega` 插件之后，就可以了.

下次启动直接容器启动即可
```bash
# 启动
$ docker start v2ray

# 停止
$ docker stop v2ray

# 重启
$ docker restart v2ray
```

### 参考
[V2Fly](https://www.v2fly.org/guide/start.html#%E5%AE%A2%E6%88%B7%E7%AB%AF)
