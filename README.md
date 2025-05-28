# ansible_keepalived
本文演示了如何使用ansible+keepalived为两台机器快速部署主从keepalived服务
# 1.安装前

## 修改变量文件group_vars/all.yml

```yaml
vim group_vars/all.yml
 
docker_data_dir: /app/docker_data
keepalived_masterIP: 192.168.2.190   #主
keepalived_slaveIP: 192.168.2.191    #从
keepalived_master_routerid: LVS_DEVEL1   #routerid，主从需要不一致
keepalived_slave_routerid: LVS_DEVEL2
keepalived_master_interface: ens33    #网卡名称，主从按实际情况填写
keepalived_slave_interface: ens33
keepalived_virtual_router_id: "51"   #router组id，主从需要一致
keepalived_master_priority: "100"   #优先级，主>从
keepalived_slave_priority: "80"
keepalived_auth_pass: "1111"   #password，主从一致
keepalived_vip: 192.168.2.100    #需要设置的VIP地址
```

## 修改主机清单

```yaml
[keepalived_master]   
192.168.2.190
[keepalived_slave]  
192.168.2.191

[keepalived_cluster:children]
keepalived_master
keepalived_slave
```

## 修改setup.sh安装脚本

```sh

export ssh_pass="sulibao"  #服务器root密码，这里要求两台台服务器密码一致

```

# 2.安装

```sh
bash setup.sh
```

# 3.验证VIP故障转移

```sh
[root@test1 ~]# ip a | grep ens33
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    inet 192.168.2.190/24 brd 192.168.2.255 scope global noprefixroute ens33
    inet 192.168.2.100/32 scope global ens33
    
[root@test2 ~]# ip a | grep ens33
2: ens33: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc pfifo_fast state UP group default qlen 1000
    inet 192.168.2.191/24 brd 192.168.2.255 scope global noprefixroute ens33
    inet 192.168.2.100/32 scope global ens33
```