# redvirt
Инструкция инсталляции системы виртуализации redvirt

```
yum update -y
hostnamectl set-hostname vlgd-node1.vlgd.redvirt
```

/etc/hosts
```
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6

172.31.40.34 vlgd-node1.vlgd.redvirt
172.31.40.36 vlgd-node2.vlgd.redvirt

172.31.40.39 hosted-engine.vlgd.redvirt
192.168.1.6 nfs1.vlgd.redvirt
```

/etc/selinux/config

```
# This file controls the state of SELinux on the system.
# SELINUX= can take one of these three values:
#     enforcing - SELinux security policy is enforced.
#     permissive - SELinux prints warnings instead of enforcing.
#     disabled - No SELinux policy is loaded.
SELINUX=disabled
# SELINUXTYPE= can take one of these three values:
#     targeted - Targeted processes are protected,
#     minimum - Modification of targeted policy. Only selected processes are protected.
#     mls - Multi Level Security protection.
SELINUXTYPE=targeted
```

### Создание сетевого интерфейса для перемычки между серварми

##### Список интерфейсов
```
ip -br link
```

/etc/sysconfig/network-scripts/ifcfg-enp59s0f0
```
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
NAME=enp59s0f0
UUID=71049054-8059-4c4e-b363-3fa327abe79e
DEVICE=enp59s0f0
ONBOOT=yes
IPADDR=192.168.1.1
NETMASK=255.255.255.248
```

##### Подготовка жесткого диска



```
lsblk
```
```
NAME        MAJ:MIN RM   SIZE RO TYPE MOUNTPOINT
sda           8:0    0 447,1G  0 disk
├─sda1        8:1    0   200M  0 part /boot/efi
├─sda2        8:2    0     1G  0 part /boot
└─sda3        8:3    0 445,9G  0 part
  ├─ro-root 254:0    0    50G  0 lvm  /
  ├─ro-swap 254:1    0     4G  0 lvm  [SWAP]
  └─ro-home 254:2    0 391,9G  0 lvm  /home
sdb           8:16   0   8,2T  0 disk
sr0          11:0    1   2,2G  0 rom  /mnt/cd
```

```
cfdisk /dev/sdb
```
