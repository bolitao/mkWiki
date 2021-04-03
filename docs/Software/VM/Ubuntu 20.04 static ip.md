``` shell
sudo vim 01-network-manager-all.yaml
```

content:

``` yml
# Let NetworkManager manage all devices on this system
network:
  version: 2
  renderer: NetworkManager
```

rewrite:

``` yml
# Let NetworkManager manage all devices on this system
network:
  version: 2
  renderer: NetworkManager
  ethernets:
    ens33:
      dhcp4: no
      addresses:
        - 192.168.79.131/24
      gateway4: 192.168.79.2
      nameservers:
          addresses: [8.8.8.8, 1.1.1.1]
```

apply:

``` bash
sudo netplan apply
```

## virtualbox?

``` yml
# Let NetworkManager manage all devices on this system
network:
  version: 2
  renderer: NetworkManager
  ethernets:
    enp0s3:
      dhcp4: no
      addresses:
        - 192.168.79.131/24
      gateway4: 192.168.79.1
      nameservers:
        addresses: [192.168.79.1, 114.114.114.114]
    enp0s8:
      dhcp4: no
      addresses: [10.0.10.101/24]
      gateway4: 10.0.10.1
      nameservers:
        addresses: [10.0.10.1, 114.114.114.114]
```
