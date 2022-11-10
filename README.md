# redvirt
Инструкция инсталляции системы виртуализации redvirt

## Подключение локального репозитория redos
```
vi /etc/yum.repos.d/local-base.repo
```
```
[base7.2]
name=RedOS - Local Base
baseurl=http://10.7.7.249/redos/redos7.2/base7.2
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-RED-SOFT
```

```
vi /etc/yum.repos.d/local-updates.repo
```
```
[updates7.2]
name=RedOS - Local Updates
baseurl=http://10.7.7.249/redos/redos7.2/updates-redos/
enabled=1
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-RED-SOFT

```

## Отключение стандартных репозиториев
##### Устанавливается enabled=0 
```
vi /etc/yum.repos.d/RedOS-Base.repo
```
```
# RedOS-Base.repo
#
# The mirror system uses the connecting IP address of the client and the
# update status of each mirror to pick mirrors that are updated to and
# geographically close to the client.  You should use this for RedOS updates
# unless you are manually picking other mirrors.
#
# If the mirrorlist= does not work for you, as a fall back you can try the
# remarked out baseurl= line instead.
#
#

[base]
name=RedOS - Base
baseurl=http://repo.red-soft.ru/redos/7.2/$basearch/os
enabled=0
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-RED-SOFT
```

```
vi  /etc/yum.repos.d/RedOS-Updates.repo
```
```
# RedOS-Base.repo
#
# The mirror system uses the connecting IP address of the client and the
# update status of each mirror to pick mirrors that are updated to and
# geographically close to the client.  You should use this for RedOS updates
# unless you are manually picking other mirrors.
#
# If the mirrorlist= does not work for you, as a fall back you can try the
# remarked out baseurl= line instead.
#
#

[updates]
name=RedOS - Updates
baseurl=http://repo.red-soft.ru/redos/7.2/$basearch/updates
enabled=0
gpgcheck=1
gpgkey=file:///etc/pki/rpm-gpg/RPM-GPG-KEY-RED-SOFT
```

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
`SELINUX=disabled`
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
##### Разметка `gpt`
##### Выбрать пункт "Новый"
![v1](https://github.com/mnmyasis/redvirt/blob/master/v1.JPG)

##### Выбрать тип "Linux LVM"
![v2](https://github.com/mnmyasis/redvirt/blob/master/v2.JPG)
##### Выбрать "Запись"
##### Выбрать "Выход"

