# VMware Workstation 下的 CentOS 使用固定 IP

## VMware Workstation

![](https://cdn.jsdelivr.net/gh/bolitao/PicRepository/img/image-20200913205546664.png)

![image-20200913205506776](https://cdn.jsdelivr.net/gh/bolitao/PicRepository/img/image-20200913205506776.png)

## windows

find the VMware Workstation NAT Network Adapter, then change the IPV4 config:

![image-20200913205419340](https://cdn.jsdelivr.net/gh/bolitao/PicRepository/img/image-20200913205419340.png)

## CentOS network config

``` shell
sudo vi /etc/sysconfig/network-scripts/ifcfg-ens33
```

content:

```shell
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=static
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=ens33
UUID=7ef0da97-7e3f-4052-85e5-e90c8386167f
DEVICE=ens33
ONBOOT=yes
IPADDR=192.168.11.6
GATEWAY=192.168.11.2
NETMASK=255.255.255.0
DNS1=192.168.11.2
```

`network.service` isn't available on CentOS8 by default, so we should use `NetworkManager.service` and restart the it:

``` shell
sudo systemctl restart NetworkManager
# or use the below command:
# sudo nmcli connection reload
```

Check either IP is changed:

```shell
ip addr
```

output:

![image-20200913210218875](https://cdn.jsdelivr.net/gh/bolitao/PicRepository/img/image-20200913210218875.png)

done!
