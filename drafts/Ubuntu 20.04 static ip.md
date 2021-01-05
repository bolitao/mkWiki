``` shell
sudo vim /etc/netplan/50-cloud-init.yaml
```

content:

``` yml
# This is the network config written by 'subiquity'
network:
  ethernets:
    enp0s3:
      dhcp4: true
  version: 2
```

rewrite:

``` yml

```
