---
title: SpinRite 6 on MacOS External Drive
date: 2020-03-16
layout: post
author: andrew
image: assets/images/apple.png
categories: [macos, "system administration"]
---

# Using SpinRite on MacOS

Primarily marketed as a Windows/DOS utility, SpinRite is capable of reading and writing each sector of a hard drive, giving the user an idea of the lifespan left in a drive. In worst-case scenarios, Spinrite has been used to recover data.

Installation and use on a Mac, however, has been tricky, and hard to use. In this post I hope to simplify and provide concise instructions on how to install, use, and scan external drives.

### Requirements

* Virtualbox (6.1)
* SpinRite.iso
* Hard Drive in need of scan

### Set up Virtualbox

1. Create a VMDK from the raw external drive
We first need to create a `.vmdk` file that will provide us RAW access to our external drive. You'll need to know the disk number (`/dev/diskX`) for the drive you want to use SpinRite on.

Create the vmdk by running the following command in your terminal:

```
sudo vboxmanage internalcommands createrawvmdk \
  -filename "/Users/$USER/Desktop/rawDisk.vmdk" \
  -rawdisk /dev/diskX
```

Be sure to replace `/dev/diskX` with your correct disk number.

2. Virtualbox must be lauched as the root/sudo user:

```
sudo /Applications/VirtualBox.app/Contents/MacOS/VirtualBox
```

This will open up the Virtualbox GUI as the root user. Note: DO NOT USE THIS METHOD for running normal Virtual Machines! SpinRite needs low-level access that only sudo can provide.

### Create a new DOS Virtual Machine
  * Click 'New'
    * Name: `Spinrite`
    * Machine Folder: `/var/root/VirtualBox VMs`
    * Type: `Other`
    * Version: `DOS`
  * Memory Size: `32MB`
  * Hard Disk: `Do not add a virtual hard disk`
    * VBox will warn you that you're creating a VM without a HDD, Click `Continue`

### Open the new VM's Settings > Storage
  * Select "Controller: IDE" and choose the icon to "Adds a Hard Disk"
    * Choose "Add" from the top, then find and open the `rawDisk.vmdk` we created earlier on your desktop.
  * Click on the "Empty" CD-ROM drive, Click on the CD-ROM icon next to "Optical Drive" on the right side. Select `Choose a disk file`
  * Select the `SpinRite.iso` file made from SpinRite.exe

<img src="https://i.imgur.com/IVKHYaD.png" width="85%" style="border:1px solid black;">

### Run the VM

When you start the VM, SpinRite should automagically see the drive. Enter your SpinRite settings and let the magic happen.
