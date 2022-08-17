本地 kubectl 远程访问 minikube
===============================

### 场景

使用 `minikube` 在 UCloud 主机搭建了一个单机 Kubernetes 集群.
> 远程服务器是使用 KVM 运行 minikube 的.

为了方便操作，配置本地使用 `kubectl` 直接连接远程的 Kubernetes 集群.

### 如何配置

复制远程服务器 `kubectl` 的配置到本地
```bash
$ scp root@{远程服务器IP} /root/.kube/config /home/h2o/.kube/config
```

复制相关证书文件到本地
```bash

$ scp root@{远程服务器IP} /root/.minikube/ca.crt /home/h2o/.kube/uhost/
$ scp root@{远程服务器IP} /root/.minikube/profiles/minikube/client.crt /home/h2o/.kube/uhost/
$ scp root@{远程服务器IP} /root/.minikube/profiles/minikube/client.crt /home/h2o/.kube/uhost/
```
> 证书文件具体路径，可以查看远程服务器 /root/.kube/config 文件

把 `certificate-authority`, `client-certificate`, `client-key` 替换为本地目录路径.
```bash
$ cat .kube/config
apiVersion: v1
clusters:
- cluster:
    certificate-authority: /home/h2o/.kube/uhost/ca.crt
    ...
    server: https://192.168.49.2:8443
  ...
- name: minikube
  user:
    client-certificate: /home/h2o/.kube/uhost/client.crt
    client-key: /home/h2o/.kube/uhost/client.key
```

这里把 `server: https://192.168.49.2:8443` 替换为 `server: https://127.0.0.1:18443`.  
因为 `192.168.49.2` 是远程服务器 `KVM` 虚拟机的网络地址，一个局域网地址，不对外暴露，本地无法建立连接.  

替换成本地之后，利用 `ssh` 隧道，本地端口转发进行访问.
```bash
$ ssh -L 18443:192.168.49.2:8443 -N -f root@{远程服务器IP}
```


最后，配置完成.
```bash
$ kubectl cluster-info
Kubernetes control plane is running at https://127.0.0.1:18443
CoreDNS is running at https://127.0.0.1:18443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.
```


### 参考
[远程访问minikube](https://cloud-atlas.readthedocs.io/zh_CN/latest/kubernetes/startup_prepare/remote_minikube.html)
