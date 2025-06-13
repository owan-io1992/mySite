---
date: 2025-06-12T02:49:00
draft: false
tags:
- Vagrant
title: vagrant introduction
---
![](images/banner.jpeg)
<!--more-->

official site: https://developer.hashicorp.com/vagrant

vagrant 是一個工具幫助管裡 virtual machines  
好處是能夠快速的用單一 source 建立多分副本  
並預先設定好內容  

適合用在地端快速設定/重建開發環境  
對於 devops/SRE 個人認為是必學工具  
畢竟開發過程中需要一次次的重複確認 能夠快速重建乾淨的環境是相當重要的  

vagrant deafult 使用 virtualbox 作為 providers(執行 VM 的 hypervisor)  
雖然也支援 hyper-v/vmware 不過因為設定多少有不同的部份 相容性也是問題  
個人是建議就乖乖使用 virtualbox 避免不必要的踩雷  
[providers](https://developer.hashicorp.com/vagrant/docs/providers/default)  

## getting start
1. install [vagrant](https://developer.hashicorp.com/vagrant/downloads)  
2. install [virtualbox](https://www.virtualbox.org)  
3. crate a Vagrantfile  

```bash
Vagrant.configure("2") do |config|
  config.vm.box = "hashicorp/bionic64"
end
```

4. provision VM  `vagrant up`
5. connect VM `vagrant ssh`
6. cleanup `vagrant destroy -f`

## tips
### create base box 
box 即為 vagrant 在建立 VM 時的 image  
vagrant 不會從 iso 開始安裝  
是將已安裝完成的 VM 打包 box file 作為 VM source  

雖然社群有現成的 box  
但在開發過程中, 會需要預先安裝好開發環境 比如說 python  
雖然可以在 vagrant 部屬過程中安裝  但那樣效率極低  
所以建議是自己先安裝好 VM  先把基本需要的東西準備好後  
包成 box 供後續使用  
詳細請見 [create base box](https://developer.hashicorp.com/vagrant/docs/providers/virtualbox/boxes)  


### network

vagrant default 會使用 nat network 對外 + 1 private network  
在建立 VM 後
預設情況使用 `vagrant ssh` 可以連進 VM   
但要配合其他工具連線就顯的不易使用, 能夠直接使用 ssh or tcp/ip 連線是最容易的    
但 private network 是 DHCP  進而導致需要知道 IP 後才能連線  
這邊只要在 Vagrantfile 中提供詳細的設定即可, 後面再一併提供範例  

### use loop  
如果一次只能建一台 VM
那使用 snapshot 功能即可  
vagrant 讓你可以建立多台 VM  
這才是用他的價值所在  
 
### use link clone 
如果要建立 10 個 VM   
每個都要 10G 的空間  
那不只建立過程會慢, 甚至浪費硬體 resource  
只要在 vagrantfile 加入 [linked-clones](https://developer.hashicorp.com/vagrant/docs/providers/virtualbox/configuration#linked-clones) 即可  


## sample

經過以上改善  
成品如下  

```bash
# -*- mode: ruby -*-
# vi: set ft=ruby :

$vms = 3 # how many VM to create
$v_cpus = 2 # set cpu source
$v_memory = 2048 # set memory source


Vagrant.configure("2") do |config|

# box source
config.vm.box = "server24.box"  # box source
# box authenticate
  config.ssh.username = "ubuntu"
  config.ssh.password = "ubuntu"

# config VM resource  
  config.vm.provider "virtualbox" do |v|
    v.linked_clone = true
    v.cpus = $v_cpus
    v.memory = $v_memory
  end

# use loop create VM
  (1..$vms).each do |i|
    # Defining VM properties
    config.vm.define "lab_vm#{i}" do |node|
      node.vm.hostname = "node#{i}.owan.lab"
      node.vm.network "private_network", ip: "192.168.56.#{100+i}"  # set VM IP
      node.vm.provision "shell",reboot: true, inline: "echo reboot"
    end
  end
end
```