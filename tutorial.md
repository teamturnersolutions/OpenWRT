![OpenWrt Logo](https://upload.wikimedia.org/wikipedia/commons/8/84/OpenWrt_Logo.svg)

## Blog Post

https://community.bigbeartechworld.com/t/setting-up-openwrt-on-a-virtual-machine-with-proxmox/257

## Get url to image
```
https://images.linuxcontainers.org/images/openwrt/23.05/amd64/default/20231123_11:57/rootfs.tar.xz
```

## Create the vm
```
pct create 202 /var/lib/vz/template/cache/OpenWRT.tar.xz --arch amd64 --hostname OpenWrt-21.02 --rootfs local-lvm:20 --memory 1024 --cores 2 --ostype unmanaged --unprivileged 1
```

## Add the lan bridge
```
vim /etc/config/network
```

```
config interface 'lan'
        option type 'bridge'
        option ifname 'eth0'
        option proto 'static'
        option netmask '255.255.255.0'
        option ipaddr '192.168.1.*'
```
