---
id: tutorial-connect-to-an-iscsi-target
summary: Mounting an iSCSI device under Ubuntu.
categories: server
tags: tutorial,nas,iscsi,networking,ubuntu,server
difficulty: 2
status: published
published: 2018-01-08
author: Daniel Lim Wee Soong <weesoong.lim@gmail.com>
feedback_url: https://github.com/canonical-websites/tutorials.ubuntu.com/issues

---

# Connect to an iSCSI target

## Overview
Duration: 1:00

iSCSI (Internet Small Computer System Interface) is a protocol that allows SCSI commands to be transmitted over a network. Typically iSCSI is implemented in a SAN (Storage Area Network) to allow servers to access a large store of hard drive space. The iSCSI protocol refers to clients as initiators and iSCSI servers as targets.

Ubuntu Server can be configured as both an iSCSI initiator and a target. This guide provides commands and configuration options to setup an iSCSI initiator. It is assumed that you already have an iSCSI target on your local network and have the appropriate rights to connect to it. The instructions for setting up a target vary greatly between hardware providers, so consult your vendor documentation to configure your specific iSCSI target.

### What you'll learn
- How to set up an iSCSI initiator to connect to an iSCSI target.

### What you'll need

- Any supported version of Ubuntu.

It is assumed that the reader already has an iSCSI target configured.

## Set up iSCSI initiator
Duration: 3:00

### Install iSCSI Initiator

To configure Ubuntu Server as an iSCSI initiator, we have to first install the `open-iscsi` package.

```
sudo apt install open-iscsi
```

### Configure iSCSI Initiator

Once the open-iscsi package is installed, edit `/etc/iscsi/iscsid.conf` changing the following:

```
node.startup = automatic
```

We can check which targets are available by using the `iscsiadm` utility. Simply enter the following in a terminal:

```
sudo iscsiadm -m discovery -t st -p <target-ip-address>
```

The flags used are to:-
1. -m: determine the mode that iscsiadm executes in.
2. -t: specify the type of discovery.
3. -p: option indicates the target IP address.

If the target is available we should see output similar to the following:

```
<target-ip-address>:3260,1 iqn.1992-05.com.emc:sl7b92030000520000-2
```

Positive
: **Note**
The iqn number and IP address above will vary depending on your hardware.

### Configure user credentials

You should now be able to connect to the iSCSI target, and depending on your target setup you may have to enter user credentials. Login to the iSCSI node:

```
sudo iscsiadm -m node --login
```

Now, we can check to make sure that the new disk has been detected using `dmesg`:

```
$ dmesg | grep sd

[    4.322384] sd 2:0:0:0: Attached scsi generic sg1 type 0
[    4.322797] sd 2:0:0:0: [sda] 41943040 512-byte logical blocks: (21.4 GB/20.0 GiB)
[    4.322843] sd 2:0:0:0: [sda] Write Protect is off
[    4.322846] sd 2:0:0:0: [sda] Mode Sense: 03 00 00 00
[    4.322896] sd 2:0:0:0: [sda] Cache data unavailable
[    4.322899] sd 2:0:0:0: [sda] Assuming drive cache: write through
[    4.323230] sd 2:0:0:0: [sda] Cache data unavailable
[    4.323233] sd 2:0:0:0: [sda] Assuming drive cache: write through
[    4.325312]  sda: sda1 sda2 < sda5 >
[    4.325729] sd 2:0:0:0: [sda] Cache data unavailable
[    4.325732] sd 2:0:0:0: [sda] Assuming drive cache: write through
[    4.325735] sd 2:0:0:0: [sda] Attached SCSI disk
[ 2486.941805] sd 4:0:0:3: Attached scsi generic sg3 type 0
[ 2486.952093] sd 4:0:0:3: [sdb] 1126400000 512-byte logical blocks: (576 GB/537 GiB)
[ 2486.954195] sd 4:0:0:3: [sdb] Write Protect is off
[ 2486.954200] sd 4:0:0:3: [sdb] Mode Sense: 8f 00 00 08
[ 2486.954692] sd 4:0:0:3: [sdb] Write cache: disabled, read cache: enabled, doesn't
 support DPO or FUA
[ 2486.960577]  sdb: sdb1
[ 2486.964862] sd 4:0:0:3: [sdb] Attached SCSI disk
```

In the example output above, sdb is the new iSCSI disk. As this is just an example, the output you see on your screen will vary.

### Mount the iSCSI disk

Last but not least, we have to create a partition, format the file system, and mount the new iSCSI disk. 
In a terminal enter:

```
sudo fdisk /dev/sdb
n
p
enter
w
```

Positive
: Note
The above commands are from inside the fdisk utility; see man fdisk for more detailed instructions. Also, the cfdisk utility is sometimes more user friendly.

Now, format the file system and mount it to `/srv`. For example:

```
sudo mkfs.ext4 /dev/sdb1
sudo mount /dev/sdb1 /srv
```

Finally, add an entry to `/etc/fstab` to mount the iSCSI drive during boot:

```
/dev/sdb1       /srv        ext4    defaults,auto,_netdev 0 0
```

## That's it!
Duration: 1:00

Congratulatons! You configured an iSCSI initiator to connect to your iSCSI target.

It is a good idea to make sure everything is working as expected by rebooting the server.

If you need some guidance with other parts of Ubuntu, take a look at the [community documentation][commdocs], and if you get stuck, help is always at hand:

* [Ask Ubuntu][askubuntu]
* [Ubuntu Forums][forums]
* [IRC-based support][ubuntuirc]

<!-- LINKS -->
[askubuntu]: https://askubuntu.com/
[forums]: https://ubuntuforums.org/
[ubuntuirc]: https://wiki.ubuntu.com/IRC/ChannelList
