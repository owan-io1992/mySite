---
date: 2025-06-13T03:30:00
draft: false
tags:
- k3s
title: k3s install advance 2
---
![alt](images/banner.png)  

<!--more-->
這一篇要導入其他 component  
- CNI: Cilium Without kube-proxy 
- ingress class: haproxy ingress 
- storage class: Longhorn


## CNI: Cilium Without kube-proxy 
Cilium 作為一個相當知名的 CNI  
他使用 eBPF 技術大大提昇了 performance  
另外也進一步取代 kube-proxy  讓網路效率再提昇  
雖然他目前也能充當 ingress class/service mesh  
但個人認為功能面還不及其他 porject  
術業有專攻, 讓他當 CNI 即可

## ingress class: haproxy ingress 
## storage class: Longhorn