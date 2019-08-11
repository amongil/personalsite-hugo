---
title: "I'm joining the Storj network"
date: 2019-08-11T11:33:59+02:00
draft: false
featuredImg: ""
tags: 
  - storj
  - ethereum
  - decentralized
  - storage
  - raspberrypi
---
![img](https://storj.io//press-kit/Storj_io-logo-white.svg)

Lately, I have not been giving much attention to my [Raspberry Pi 3 B+](https://www.raspberrypi.org/products/raspberry-pi-3-model-b-plus/) and I remembered that a few months back I had received an invitation to the [Storj network](https://storj.io/). Luckily, I have a spare [1TB external HDD](https://www.amazon.com/Seagate-Portable-External-Hard-Drive/dp/B07CRG7BBH/ref=sr_1_3?keywords=STGX1000400&qid=1565514627&s=gateway&sr=8-3) that I can now hopefully put to good use.

>Storj is an S3-compatible platform and suite of decentralized applications that allows you to store data in a secure and decentralized manner.[^1]
---

# Node installation

After restoring my Pi to factory defaults and enabling SSH to connect and run the necessary installation commands from my Macbook, it is time to start with the prerequisites: mounting the external hard drive, installing docker and setting up port-forwarding.

## Prerequisites

### Mounting the external hard drive

My HDD came formatted as NTFS and with a bunch of vendor files that I want to get rid of and reformat as *ext4*.

For that, I make use of the *fdisk* utility to create the partitions of type *Linux*:

```bash
pi@raspberrypi:~ sudo fdisk /dev/sda

Welcome to fdisk (util-linux 2.29.2).
Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.


Command (m for help): p
Disk /dev/sda: 931,5 GiB, 1000204885504 bytes, 1953525167 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xe35c7600

Device     Boot Start        End    Sectors   Size Id Type
/dev/sda1  *       64 1953520128 1953520065 931,5G  7 HPFS/NTFS/exFAT

Command (m for help): d
Selected partition 1
Partition 1 has been deleted.

Command (m for help): p
Disk /dev/sda: 931,5 GiB, 1000204885504 bytes, 1953525167 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xe35c7600

Command (m for help): n
Partition type
   p   primary (0 primary, 0 extended, 4 free)
   e   extended (container for logical partitions)
Select (default p): p
Partition number (1-4, default 1): 1
First sector (2048-1953525166, default 2048):
Last sector, +sectors or +size{K,M,G,T,P} (2048-1953525166, default 1953525166):

Created a new partition 1 of type 'Linux' and of size 931,5 GiB.
```

And then formatting the partition as *ext4*:

```bash
pi@raspberrypi:~ $ sudo mkfs.ext4 /dev/sda1 -L storjspace01
mke2fs 1.43.4 (31-Jan-2017)
Se está creando un sistema de ficheros con 244190389 bloques de 4k y 61054976 nodos-i
UUID del sistema de ficheros: 8cf9d53a-7ec4-4a7b-a5b0-ad5ac7d272b8
Respaldo del superbloque guardado en los bloques:
	32768, 98304, 163840, 229376, 294912, 819200, 884736, 1605632, 2654208,
	4096000, 7962624, 11239424, 20480000, 23887872, 71663616, 78675968,
	102400000, 214990848

Reservando las tablas de grupo: hecho
Escribiendo las tablas de nodos-i: hecho
Creando el fichero de transacciones (262144 bloques): hecho
Escribiendo superbloques y la información contable del sistema de ficheros: hecho
```

Now we can mount it:

```bash
pi@raspberrypi:~ $ sudo mkdir /mnt/storjspace01
sudo mkdir /storjspace01

pi@raspberrypi:~ $ sudo mount /dev/sda1 /mnt/storjspace01
sudo mount /dev/sda1 /storjspace01
pi@raspberrypi:~ $ df -h
S.ficheros     Tamaño Usados  Disp Uso% Montado en
/dev/root         28G   4,0G   23G  15% /
devtmpfs         459M      0  459M   0% /dev
tmpfs            464M      0  464M   0% /dev/shm
tmpfs            464M    13M  451M   3% /run
tmpfs            5,0M   4,0K  5,0M   1% /run/lock
tmpfs            464M      0  464M   0% /sys/fs/cgroup
/dev/mmcblk0p6    71M    23M   49M  32% /boot
tmpfs             93M      0   93M   0% /run/user/1000
/dev/mmcblk0p5    30M   398K   28M   2% /media/pi/SETTINGS
/dev/sda1        916G    77M  870G   1% /mnt/storjspace01
```

Let's now modify the *fstab* file to define the location where the storage device will be automatically mounted when the Raspberry Pi starts up, to persist the configuration across reboots. I have to add the following line to the */etc/fstab* file:

```
/dev/sda1 /mnt/storjspace01 ext4 defaults 0 0
```

### Installing Docker

I will make use of the official Docker installation script:

```bash
$ curl -fsSL https://get.docker.com -o get-docker.sh
```

Once downloaded, run it:

```bash
$ sh get-docker.sh
```

### Setting up port forwarding

Here I have to go to my router settings and set up port forwarding for port 28967 for the local ip of the Raspberry Pi on my LAN.

## Setup

### Create an identity

This is supposed to take a good bit of computing power so I will create this identity on my Macbook, sign it and then transfer it to my Raspberry Pi.

After downloading the binary for macOS:

```bash
./identity_darwin_amd64 create storagenode
```

This will create an identity that I will sign with the token that I have received via email back when I signed up for the alpha:

```bash
./identity_darwin_amd64 authorize storagenode $MYTOKEN
```

> Make sure to back up this file as it uniquely identifies you within the network

 Now let's transfer it to the Pi:
```
$ scp -r ~/Library/Application\ Support/Storj/Identity/storagenode/ pi@192.168.1.42:/home/pi/.local/share/storj/identity/storagenode/
pi@192.168.1.42's password:
ca.1565521730.cert                                                                                                                                                        100%  546   103.2KB/s   00:00
ca.cert                                                                                                                                                                   100% 1076   170.0KB/s   00:00
identity.cert                                                                                                                                                             100% 1618    12.4KB/s   00:00
identity.key                                                                                                                                                              100%  241    43.4KB/s   00:00
identity.1565521730.cert                                                                                                                                                  100% 1088   170.1KB/s   00:00
ca.key
```

### Download and run the storagenode container

On the Pi SSH session:

```bash
docker pull storjlabs/storagenode:arm
```

```bash
docker run -d --restart unless-stopped -p 28967:28967 \
    -e WALLET="0xxxxxxxxxx" \
    -e ADDRESS="myaddress:28967" \
    -e BANDWIDTH="2TB" \
    -e STORAGE="800GB" \
    --mount type=bind,source="/home/pi/.local/share/storj/identity/storagenode",destination=/app/identity \
    --mount type=bind,source="/mnt/storjspace01",destination=/app/config \
    --name storagenode storjlabs/storagenode:arm
```

OK, let's take a look at the logs and see if everything is alright:

```
$ docker logs  storagenode
2019-08-11T11:57:06.494Z	INFO	Node started
2019-08-11T11:57:06.495Z	INFO	Public server started on [::]:28967
2019-08-11T11:57:06.495Z	INFO	Private server started on 127.0.0.1:7778
2019-08-11T11:57:06.496Z	INFO	vouchers	Checking vouchers
2019-08-11T11:57:06.500Z	INFO	piecestore:monitor	Remaining Bandwidth	{"bytes": 2000000000000}
2019-08-11T11:57:06.501Z	INFO	bandwidth	Performing bandwidth usage rollups
2019-08-11T11:57:06.502Z	INFO	vouchers	Requesting voucher	{"satellite": "12EayRS2V1kEsWESU9QMRseFhdxYxKicsiFmxrsLZHeLUtdps3S"}
2019-08-11T11:57:06.503Z	INFO	vouchers	Requesting voucher	{"satellite": "118UWpMCHzs6CvSgWd9BfFVjw5K9pZbJjkfZJexMtSkmKxvvAW"}
2019-08-11T11:57:06.506Z	INFO	vouchers	Requesting voucher	{"satellite": "121RTSDpyNZVcEU84Ticf2L1ntiuUimbWgfATz21tuvgk3vzoA6"}
2019-08-11T11:57:06.508Z	INFO	vouchers	Requesting voucher	{"satellite": "12L9ZFwhzVpuEKMUNUqkaTLGzwY9G24tbiigLiXpmZWKwmcNDDs"}
2019-08-11T11:57:06.621Z	INFO	running on version v0.16.1
2019-08-11T11:57:06.964Z	INFO	vouchers	Voucher request denied. Vetting process not yet complete
2019-08-11T11:57:07.125Z	INFO	vouchers	Voucher request denied. Vetting process not yet complete
2019-08-11T11:57:07.396Z	INFO	vouchers	Voucher request denied. Vetting process not yet complete
2019-08-11T11:57:08.018Z	INFO	vouchers	Voucher request denied. Vetting process not yet complete
```

As we can see from the logs, the node is attempting to join the network. But there is something called *vetting process* which is preventing it from.

>By default, storage nodes will accept requests to store data from all Satellites on the network. When a new Satellite comes online and makes the first request to a storage node, there will be a vetting process to limit exposure and build trust. Storage nodes can indicate the maximum amount of data they will store for untrusted Satellites. This is a two-way street. When a new storage node comes online, the Satellite vets the new storage node as well to ensure it can reliability store data. Over time, the storage node will build up its reputation with the Satellite and will be allowed to store more data. [^2]

---

# Conclussions

Preparing your Raspberry Pi and your HDD to work with the Storj network is a pretty straight-forward job. Storj is gaining some traction and now is offering an Enterprise version of the network called [Tardigrade](https://tardigrade.io/). I imagine as Storj takes off more and more powerful nodes will join the network and it might even be the case now that my poor Raspberry Pi is no competition to most of the nodes. Anyways, the process was fun and my hopes are still up. Hope you enjoyed following the process alongside me. Until next time!


[^1]: From https://documentation.storj.io
[^2]: From https://storj.io/blog/2019/04/what-storage-node-operators-need-to-know-about-satellites/