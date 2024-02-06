---
title: Proxmox - Migrate LVM to ZFS
date: 2024-02-06
layout: post
author: andrew
image: assets/images/proxmox.jpg
categories: [linux, homelab, "system administration"]
---

# Migrate LVM/PVG to ZFS in Proxmox

Let's imagine a scenario: you set up a proxmox server a long time ago. You didn't really know what you were doing and just clicked "Next, Next, Next". Now the time has come that your proxmox server drive is full and you need to properly set things up. Using ZFS is a simple way to create RAID backups, so that if one of your disk drives goes down, the data is redundently baked up across one or more disks. Lucky for us, Proxmox makes the ZFS setup a breeze. Migrating from your old LVM to the new ZFS drive, however, can be a pain. Let's figure out how to migrate your LVM containers/VMs to your new ZFS drives.

## ZFS Setup

First, we need to set up our new ZFS drives. ZFS is a disk format much like NTFS or EXT4. In the proxmox UI, go to your server, Disks >> ZFS. Inside of this menu click `Create: ZFS`. 

You can name it anything you'd like, but I'll call mine `zfs-tank`. Uncheck `Add Storage` and set your `RAID Level`. Use `lz4` compression.

Check your disk to add it to the pool and click create. Easy. You've created your first ZFS pool.

## ZFS Pools

Now that we've got disk space, let's create some "pools".  Pools are logical groupings of storage. We'll create two pools, "Backups" and "VM Drives."

From the console of your proxmox server, create the pools:

```bash
root@pve# zfs list
NAME                 USED  AVAIL     REFER  MOUNTPOINT
zfs-tank            33.8G  1.72T      104K  /zfs-tank

root@pve# zfs create zfs-tank/vm-drives
root@pve# zfs create zfs-tank/backups
```

Pretty simple, right? Now we should have one large tank, and two pools.

```bash
zfs list
NAME                 USED  AVAIL     REFER  MOUNTPOINT
zfs-tank            33.8G  1.72T      104K  /zfs-tank
zfs-tank/backups    14.5G  1.72T     14.5G  /zfs-tank/backups
zfs-tank/vm-drives  19.3G  1.72T     19.3G  /zfs-tank/vm-drives
```

Look at the root of your drive. You should see a folder with the name of your zfs tank.

```bash
root@pve# ls -l /
total 61
lrwxrwxrwx   1 root root     7 Apr 27  2021 bin -> usr/bin
[ ... ]
drwxr-xr-x  11 root root  4096 Apr 27  2021 var
drwxr-xr-x   4 root root     4 Feb  6 15:38 zfs-tank
```

You can see that my tank, named `zfs-tank` is at the root. Inside should be the two pools you created:

```
root@pve# ls -l /zfs-tank/
total 1
drwxr-xr-x 4 root root 4 Feb  6 15:42 backups
drwxr-xr-x 5 root root 5 Feb  6 15:41 vm-drives
```

## Creating Directories

The next step is to create "Directories" for storage inside of proxmox. These directories allow you to store different types of data. We'll use our pools to store Backups and VM/Container Drive images.

From the Proxmox UI, click your `Datacenter`, then go into Storage. Click Add >> Directory

For ID, make up a useful name, like the name of the pool itself. For the path, enter the path to that pool. (i.e. `/zfs-tank/backups`).

For backups, Add `VZDump backup file` and `Snippets` as the Content
For disk images, add `Disk Image` and `Container` as the contents.

Make directories for all of your pools.

# Migrating from LVM to ZFS

Now comes the tricky part. How do you move your VM/Container that already has a VM Drive stored on the LVM onto the ZFS? Luckly proxmox makes this pretty simple with it's built in Backup & Restore process. This is not a 100% uptime process, but it doesn't take long.

In the Proxmox UI, find your VM/Container that you want to move. Stop it. Then click on the "Backup" tab and "Backup Now" button. Select your new ZFS backup directory as the "Storage", then click "Backup". Let it process.

To restore, click on the backup file, then "Restore". Be sure to restore to your ZFS VM Drive directory under "Storage". Click "Restore" to start the process.

That's it. When the restore is done, you should see that the VM is now using the ZFS disk as it's storage under "Hardware"

Repeat these steps for as many VMs/Containers as you need.
