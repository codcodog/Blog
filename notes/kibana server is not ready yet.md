kibana server is not ready yet
================================

### 场景

在本地安装了 `kibana`，配置连接服务器A上的 `Elasticsearch`，结果一直报 `kibana server is not ready yet`


### 方案

需要手动删除 `kibana` 相关的索引

登录服务器A
```bash
$ curl -X DELETE http://localhost:9200/.kibana*
```

### 参考
[Kibana server is not ready yet](https://stackoverflow.com/questions/58011088/kibana-server-is-not-ready-yet)
