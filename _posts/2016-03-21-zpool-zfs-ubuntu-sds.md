---
layout: post
title:  "ZFS RAIDZ + SSD Caching"
date:   2016-04-06
excerpt: "Optimal performance of KVM virtual machines via ZFS hybrid software-defined storage."
project: true
tag:
- zfs 
- nvme
- cache
- raidz
- zpool
comments: true
---

![Moon Homepage](https://cloud.githubusercontent.com/assets/754514/14509720/61c61058-01d6-11e6-93ab-0918515ecd56.png)    
    
<center><b>ZFS RAIDZ + SSD Caching on Ubuntu</b></center>
     
 For this system, I want a RAIDZ-1 pool with both an L2ARC cache and a ZIL log. It is important to note that RAIDZ-1 is NOT RAID-1, it is a special version of RAID meant for ZFS that is comparable to RAID5. RAIDZ-1 arrays can lose one disk before losing any data, and require a minimum of 3 disks. RAIDZ-2 arrays can lose two disks before losing any data, and require a minimum of 4 disks.

~~~~~
# Install prereqs
apt install -y ceph zfsutils-linux samba-common-bin samba samba-common python-glade2 system-config-samba

# clean up disks
dd if=/dev/zero of=/dev/sdb bs=1M count=200
ceph-disk zap /dev/sdb
dd if=/dev/zero of=/dev/sdc bs=1M count=200
ceph-disk zap /dev/sdc
dd if=/dev/zero of=/dev/sdd bs=1M count=200
ceph-disk zap /dev/sdd
dd if=/dev/zero of=/dev/sde bs=1M count=200
ceph-disk zap /dev/sde
dd if=/dev/zero of=/dev/sdf bs=1M count=200
ceph-disk zap /dev/sdf
dd if=/dev/zero of=/dev/sdg bs=1M count=200
ceph-disk zap /dev/sdg
dd if=/dev/zero of=/dev/sdh bs=1M count=200
ceph-disk zap /dev/sdh
dd if=/dev/zero of=/dev/sdi bs=1M count=200
ceph-disk zap /dev/sdi
dd if=/dev/zero of=/dev/sdj bs=1M count=200
ceph-disk zap /dev/sdj
dd if=/dev/zero of=/dev/sdk bs=1M count=200
ceph-disk zap /dev/sdk
dd if=/dev/zero of=/dev/sdl bs=1M count=200
ceph-disk zap /dev/sdl
dd if=/dev/zero of=/dev/sdm bs=1M count=200
ceph-disk zap /dev/sdm
dd if=/dev/zero of=/dev/sdn bs=1M count=200
ceph-disk zap /dev/sdn
dd if=/dev/zero of=/dev/sdo bs=1M count=200
ceph-disk zap /dev/sdo
dd if=/dev/zero of=/dev/sdp bs=1M count=200
ceph-disk zap /dev/sdp
dd if=/dev/zero of=/dev/sdq bs=1M count=200
ceph-disk zap /dev/sdq
dd if=/dev/zero of=/dev/sdr bs=1M count=200
ceph-disk zap /dev/sdr
dd if=/dev/zero of=/dev/sds bs=1M count=200
ceph-disk zap /dev/sds
dd if=/dev/zero of=/dev/sdt bs=1M count=200
ceph-disk zap /dev/sdt
dd if=/dev/zero of=/dev/sdu bs=1M count=200
ceph-disk zap /dev/sdu
dd if=/dev/zero of=/dev/sdv bs=1M count=200
ceph-disk zap /dev/sdv
dd if=/dev/zero of=/dev/sdw bs=1M count=200
ceph-disk zap /dev/sdw
dd if=/dev/zero of=/dev/nvme0n1 bs=1M count=200
ceph-disk zap /dev/nvme0n1
dd if=/dev/zero of=/dev/nvme1n1 bs=1M count=200
ceph-disk zap /dev/nvme1n1

# create new zfs pool
/sbin/zpool create -f -o ashift=12 zfspool raidz /dev/sdb /dev/sdc /dev/sdd /dev/sde /dev/sdf /dev/sdg /dev/sdh /dev/sdi /dev/sdj /dev/sdk /dev/sdl /dev/sdm /dev/sdn /dev/sdo /dev/sdp /dev/sdq /dev/sdr /dev/sds /dev/sdt /dev/sdu /dev/sdv /dev/sdw

# add nvme cache drives
/sbin/zpool add -f zfspool cache nvme0n1 nvme1n1

# make the media share
mkdir /zfspool/media

# create a samba group called smbgroup for the share
sudo addgroup smbgroup

# add the user to the group
sudo usermod -aG smbgroup force

# set SMB user password
sudo smbpasswd -a force

# set perms on new share
cd /zfspool/
sudo chown -R root:smbgroup media
sudo chmod -R 0770 media

# edit smb.conf
nano /etc/samba/smb.conf

# [Protected]
#  path = /samba/protected
#  valid users = @smbgroup
#  guest ok = no
#  writable = yes
#  browsable = yes
~~~~~
