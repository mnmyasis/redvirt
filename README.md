# redvirt
Инструкция инсталляции системы виртуализации redvirt

- [Подготовка серверов](#s1)
- [Установка виртуализации на мастер ноду](#s2)
- [Подключение второй ноды к кластеру](#s3)
 
<a name="s1"></a>
# Предварительная подготовка нод
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
##### Устанавливается `enabled=0` 
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
##### Проверяем подключение репозиториев
```
yum repolist
```
```
Загружены модули: fastestmirror, langpacks, vdsmupgrade
base7.2                             RedOS - Local Base                    28 434
updates7.2                          RedOS - Local Updates                 4 973
repolist: 34 354
```

## Обновление пакетов
```
yum update -y
```
## Установка hostname
```
hostnamectl set-hostname vlgd-node1.vlgd.redvirt
```

/etc/hosts
```
172.31.40.34 vlgd-node1.vlgd.redvirt
172.31.40.36 vlgd-node2.vlgd.redvirt

172.31.40.39 hosted-engine.vlgd.redvirt
192.168.1.6 nfs1.vlgd.redvirt
```

## Отключение SELINUX

```
setenforce 0
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
ip -br link show
```
```
enp59s0f0        UP             80:61:5f:13:2a:f4 <BROADCAST,MULTICAST,UP,LOWER_UP>
enp59s0f1        DOWN           80:61:5f:13:2a:f5 <NO-CARRIER,BROADCAST,MULTICAST,UP>
enp134s0f0       DOWN           80:61:5f:13:28:42 <NO-CARRIER,BROADCAST,MULTICAST,UP>
enp134s0f1       UP             80:61:5f:13:28:43 <BROADCAST,MULTICAST,UP,LOWER_UP>
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
```
systemctl disable multipathd
```
```
/etc/multipath/conf.d/server.conf
```
```
blacklist {
       devnode "^sd[a-z]"
}
```
------


<a name="s2"></a>
# Установка виртуализации на node1(далее мастер нода)
##### Не забудьте перед этим подключить диск с образом в виртуальный привод
```
mkdir -p /mnt/cd
mount /dev/cdrom /mnt/cd
cd /mnt/cd
./install.run
```
![v7](https://github.com/mnmyasis/redvirt/blob/master/v7.JPG)
![v8](https://github.com/mnmyasis/redvirt/blob/master/v8.JPG)
![v9](https://github.com/mnmyasis/redvirt/blob/master/v9.JPG)
![v10](https://github.com/mnmyasis/redvirt/blob/master/v10.JPG)

## Подключение репозитория Epel
```
/etc/yum.repos.d/epel.repo
```
```
[epel]
name=epel local
baseurl=http://10.7.7.249/epel/7/x86_64/
enabled=0
gpgcheck=0
```
## Инсталляция пакетов
```
yum install -y drbd drbd-pacemaker policycoreutils-python-utils pacemaker pcs psmisc
```
## Отключение репозитория Epel
```
yum-config-manager --disable epel
```
##### Инсталляция пакетов NFS
```
yum install -y nfs-utils nfs4-acl-tools
```

# Настройка DRBD

```
semanage permissive -a drbd_t
modprobe 8021q
```
```
vi /etc/drbd.d/storage.res
```
```
resource storage {
 protocol C;
 meta-disk internal;
 device /dev/drbd1;
 syncer {
  verify-alg sha1;
 }
 net {
  after-sb-0pri discard-younger-primary;
  after-sb-1pri consensus;
}
disk {
  c-fill-target 10M;
  c-max-rate   700M;
  c-plan-ahead    7;
  c-min-rate     4M;
 }

 on vlgd-node1.vlgd.redvirt {
  disk   /dev/sdb1;
  address  192.168.1.1:7789;
 }
 on vlgd-node2.vlgd.redvirt {
  disk   /dev/sdb1;
  address  192.168.1.2:7789;
 }
}
```
```
drbdadm create-md storage
```
```
drbdadm up storage
```
```
drbdadm primary --force storage
```
```
mkdir –p /storage/hdd
groupadd kvm -g 36
useradd vdsm -u 36 -g 36
chown -R 36:36 /storage/hdd
chmod 0755 /storage/hdd
mkfs -t ext4 /dev/drbd1
```

## Настройка Pacemaker

```
systemctl enable pcsd
systemctl start pcsd
systemctl enable pacemaker
```

##### Установить пароль для пользователя hacluster
```
passwd hacluster
```
```
pcs host auth node1.vlgd.redvirt
pcs cluster setup VMCluster node1.vlgd.redvirt
pcs cluster start –all
```
```
pcs property set stonith-enabled=false
pcs property set no-quorum-policy=ignore
crm_verify –L
pcs resource create ClusterIP1 ocf:heartbeat:IPaddr2 ip=192.168.1.6 cidr_netmask=29 op monitor interval=30s
pcs cluster cib drbd_cfg
pcs -f drbd_cfg resource create VMData ocf:linbit:drbd drbd_resource=storage op monitor interval=60s
pcs -f drbd_cfg resource promotable VMData promoted-max=1 promoted-node-max=1 clone-max=2 clone-node-max=1 notify=true
pcs cluster cib-push drbd_cfg --config
```
```
echo drbd >/etc/modules-load.d/drbd.conf
```
```
pcs cluster cib fs_cfg 
pcs -f fs_cfg resource create StorageFS Filesystem device="/dev/drbd1" directory="/storage/hdd" fstype="ext4"
pcs -f fs_cfg constraint colocation add StorageFS with VMData-clone INFINITY with-rsc-role=Master
pcs -f fs_cfg constraint order promote VMData-clone then start StorageFS
pcs cluster cib-push fs_cfg --config
pcs -f fs_cfg constraint
```
```
pcs resource create NfsServer ocf:heartbeat:nfsserver nfs_ip=nfs1.vlgd.redvirt nfs_shared_infodir=/nfsshare/nfsinfo op monitor interval="60"
pcs resource create NfsStorage ocf:heartbeat:exportfs directory="/storage/hdd" options="rw,no_root_squash,anongid=36,anonuid=36" clientspec="*" fsid="1" op monitor interval="60"
```
```
pcs constraint colocation add NfsServer with ClusterIP1 INFINITY
pcs constraint colocation add ClusterIP1 with NfsStorage INFINITY
pcs constraint colocation add ClusterIP1 with StorageFS INFINITY
```
```
pcs constraint order ClusterIP1 then NfsServer 
pcs constraint order NfsServer then NfsStorage
pcs constraint order StorageFS then ClusterIP1
```

## Правила firewall
```
firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="192.168.1.1" port port="7789" protocol="tcp" accept'
firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="192.168.1.2" port port="7789" protocol="tcp" accept'
firewall-cmd --permanent --add-service=high-availability
firewall-cmd --permanent --add-service=nfs
firewall-cmd --permanent --add-service=mountd
firewall-cmd --permanent --add-service=rpc-bin
firewall-cmd --zone=public --add-port=161/udp --permanent
firewall-cmd --zone=public --add-port=161/tcp --permanent
firewall-cmd --zone=public --add-port=162/udp --permanent
firewall-cmd --zone=public --add-port=162/tcp --permanent
firewall-cmd –reload
```


## Деплой виртуализации

##### В браузере перейти на страницу по адресу, который был показан вовремя инсталляции виртуализации
![v10](https://github.com/mnmyasis/redvirt/blob/master/v10.JPG)
![v3](https://github.com/mnmyasis/redvirt/blob/master/v3.JPG)
![v4](https://github.com/mnmyasis/redvirt/blob/master/v4.JPG)
![v5](https://github.com/mnmyasis/redvirt/blob/master/v5.JPG)

------


<a name="s3"></a>
# Подключение второй ноды к кластеру

##### Выбрать подключения репозитория
```
mkdir -p /mnt/cd
mount /dev/cdrom /mnt/cd
cd /mnt/cd
./install.run
```
![v9](https://github.com/mnmyasis/redvirt/blob/master/v9.JPG)
![v8](https://github.com/mnmyasis/redvirt/blob/master/v8.JPG)
![v11](https://github.com/mnmyasis/redvirt/blob/master/v11.JPG) 

## Подключение ноды
![v12](https://github.com/mnmyasis/redvirt/blob/master/v12.jpg) 
![v13](https://github.com/mnmyasis/redvirt/blob/master/v13.JPG) 
![v14](https://github.com/mnmyasis/redvirt/blob/master/v14.JPG) 
![v15](https://github.com/mnmyasis/redvirt/blob/master/v15.JPG) 


## Подключение репозитория Epel
```
/etc/yum.repos.d/epel.repo
```
```
[epel]
name=epel local
baseurl=http://10.7.7.249/epel/7/x86_64/
enabled=0
gpgcheck=0
```
## Инсталляция пакетов
```
yum install -y drbd drbd-pacemaker policycoreutils-python-utils pacemaker pcs psmisc
```
## Отключение репозитория Epel
```
yum-config-manager --disable epel
```
## Инсталляция пакетов NFS
```
yum install -y nfs-utils nfs4-acl-tools
```

# Настройка DRBD

```
semanage permissive -a drbd_t
modprobe 8021q
```
##### С мастер ноды копируем конфиг по sftp на подключаемую ноду.
```
scp /etc/drbd.d/storage.res root@192.168.1.2:/etc/drbd.d/
```
##### Продолжаем настройки на подключаемой ноде
```
drbdadm create-md storage
```
```
drbdadm up storage
```
```
mkdir –p /storage/hdd
groupadd kvm -g 36
useradd vdsm -u 36 -g 36
chown -R 36:36 /storage/hdd
chmod 0755 /storage/hdd
mkfs -t ext4 /dev/drbd1
```

## Настройка Pacemaker

```
systemctl enable pcsd
systemctl start pcsd
systemctl enable pacemaker
```

##### Установить пароль для пользователя hacluster
```
passwd hacluster
```

## Правила firewall
```
firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="192.168.1.1" port port="7789" protocol="tcp" accept'
firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="192.168.1.2" port port="7789" protocol="tcp" accept'
firewall-cmd --permanent --add-service=high-availability
firewall-cmd --permanent --add-service=nfs
firewall-cmd --permanent --add-service=mountd
firewall-cmd --permanent --add-service=rpc-bin
firewall-cmd --zone=public --add-port=161/udp --permanent
firewall-cmd --zone=public --add-port=161/tcp --permanent
firewall-cmd --zone=public --add-port=162/udp --permanent
firewall-cmd --zone=public --add-port=162/tcp --permanent
firewall-cmd –reload
```

##### Подключаемая нода
```
pcs host auth vlgd-node2.vlgd.redvirt vlgd-node1.vlgd.redvirt
```

##### Мастер нода
```
pcs host auth vlgd-node2.vlgd.redvirt
pcs cluster node add node2.rst.redvirt --start –enable
```
