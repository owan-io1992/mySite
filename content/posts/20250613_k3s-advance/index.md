---
date: 2025-06-13T00:00:00
draft: false
tags:
- k3s
title: k3s advance
---
![alt](images/banner.png)  

<!--more-->

## args in install script
在安裝時,藉由設定參數來指定安裝內容, 比如說版本  
官方有幾種設定模式  
請參考 [configuration-with-install-script](https://docs.k3s.io/installation/configuration#configuration-with-install-script)  

支援設定  
[env-variables](https://docs.k3s.io/reference/env-variables)  

INSTALL_K3S_EXEC 支援參數  

[server](https://docs.k3s.io/cli/server)  
[agent](https://docs.k3s.io/cli/agent)  

## 常用 args 說明
### specific version
在維護 cluster 中，版本一致相當重要的  
先到 [release note](https://docs.k3s.io/release-notes/v1.33.X) 找到 k3s 版本訊息  

安裝時加上 `INSTALL_K3S_VERSION` 即可  

```bash
export INSTALL_K3S_VERSION=v1.31.6+k3s1
curl -sfL https://get.k3s.io | sh -s -
```

### server/agent
[doc](https://docs.k3s.io/quick-start#install-script)    
預設是安裝 server  
先在 node1 執行  
```bash
# in node1
curl -sfL https://get.k3s.io | sh -
```

當 server 起來後, 拿到 k3s token  
```bash
# in node1
sudo cat /var/lib/rancher/k3s/server/node-token
```

node2 設定為 agent (worker node)  
```bash
# in node2
export K3S_TOKEN=<token>
export K3S_URL=https://<node1 ip>:6443
export INSTALL_K3S_EXEC="agent"
curl -sfL https://get.k3s.io | sh -s -
```

### Service Load Balancer
k8s 本身支援 loadbalancer 跟 pod 溝通, 但並沒有 implement loadbalancer  
也就是需要自行安裝 loadbalancer  
k3s 有提供 [ServiceLB](https://docs.k3s.io/networking/networking-services?_highlight=traefik#service-load-balancer) 給 on-premise 環境使用  
不過如果你想使用 MetalLB 的話 也能在安裝時加上 `--disable=servicelb`  

不過更多時候 我個人習慣是將 ingress controller 使用 host network 去達成目的   
更加簡單高效  

### ingress controller
k3s 預設安裝 [Traefik](https://docs.k3s.io/networking/networking-services?_highlight=traefik#traefik-ingress-controller)  
如果想用 haproxy or nginx 可以安裝時加上 `--disable=traefik`  


### High Availability Embedded etcd
default 情況下  
k3s 為 single server/multi client 架構  
也就是沒有 HA 的情況  
官方有提供 [High Availability Embedded etcd](https://docs.k3s.io/datastore/ha-embedded)   
只要在安裝時加上 `--cluster-init` 即可  

## k3s cluster with HA
最後結合以上  
來建立一個高可用性節點  
高可用性節點使用 KeepAlived 設定一個 virtual ip 給 Kubernetes API server 使用  
這樣在任何一 server node 死掉情況下, cluster 還能持續 control  

lab 環境採用 ubuntu 24.04  
server ip: 192.168.56.101~103  
server virtual ip: 192.168.56.100  

install keepalived  
```bash
sudo apt update && sudo apt install -y keepalived
```

Add the following to /etc/keepalived/keepalived.conf on 3 server node  
```bash
sudo tee /etc/keepalived/keepalived.conf <<EOF
global_defs {
  enable_script_security
  script_user root
}

vrrp_instance haproxy-vip {
    interface enp0s8 # change as you need
    state BACKUP
    priority 100

    virtual_router_id 51

    virtual_ipaddress {
        192.168.56.100/24
    }
}
EOF
```

restart keepalived and check  
```bash
sudo systemctl restart keepalived

ubuntu@node1:~$ ip a show dev enp0s8
3: enp0s8: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 08:00:27:4b:9e:3d brd ff:ff:ff:ff:ff:ff
    inet 192.168.56.101/24 brd 192.168.56.255 scope global enp0s8
       valid_lft forever preferred_lft forever
    inet 192.168.56.100/24 scope global secondary enp0s8
       valid_lft forever preferred_lft forever
    inet6 fe80::a00:27ff:fe4b:9e3d/64 scope link 
       valid_lft forever preferred_lft forever
```

now install 3 server node  
```bash
# in first node
export INSTALL_K3S_VERSION=v1.32.5+k3s1
export INSTALL_K3S_EXEC="server --disable=traefik --write-kubeconfig-mode 644 --cluster-init --tls-san=192.168.56.100"
curl -sfL https://get.k3s.io | sh -s -

sudo cat /var/lib/rancher/k3s/server/node-token


export INSTALL_K3S_VERSION=v1.32.5+k3s1
export INSTALL_K3S_EXEC="server --disable=traefik --write-kubeconfig-mode 644 --server https://192.168.56.101:6443 --tls-san=192.168.56.100"
export K3S_TOKEN=<fill me>
curl -sfL https://get.k3s.io | sh -s -
```

這樣就完成 k3s cluster with HA 了  
下一篇再完成 ingress class/storage class/CNI 等基本設定  

