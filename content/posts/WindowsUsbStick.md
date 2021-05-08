---
title: "Windows Server 2019 installation from USB Stick"
subtitle: "spinning discs will die."
date: 2020-02-14T00:55:13+01:00
draft: false
tags: ["WindowsServer", "Windows", "Legacy", "Diskpart", "USB-Stick", "Boot"]
categories: [windows, itstuff]

aliases:
    - /posts/WindowsUsbStick/
---

If you have to install Windows Server 2019 manually, you will come to the point where you ask yourself how you should install a ISO without burning the image to a physical disk and search this damned USB 2.0 External DVD drive which is covered by cobwebs in the cellar?

The Prerequisites are plain simple

- windows Iso file
- USB stick (at least 8GB)
<!--more-->
## Step 1: Format the Stick

Open a privileged PowerShell or Command line

``` cmd
> diskpart

# list all disks
DISKPART> list disk

# Select your USB drive which can be identified by the size in most cases
DISKPART> select disk TheNumberOfYourUSBDisk

# Throw away all content of the Disk
DISKPART> clean

# Create a primary partition
DISKPART > create partition primary

#select the partition and mark it as active
DISKPART > select partition 1
DISKPART > active

# If your USB stick is for old Hardware which at some points does not support extfat for EFI boot
# you should replace extfat with fat32 and we have some addidional work to do
DISKPART > format fs=exfat quick label="Win2019"

# Exit diskpart
DISKPART > exit

```

## Step 2: make the drive bootable

That's really easy Microsoft provides some tooling for this.
All we need is the drive letter of your USB stick (In this sample I use U:)

`bootsect /nt60 U:` will create the needed start code

## Step 3: Moving files

Mount the ISO File to your System (In this sample it is Mounted to W:)

If you are lucky and you can use extfat you can just move alle files with xcopy and you are done
`xcopy.exe W:\*.* U:\ /E /H /F`

and you are done.

If you have to use Legacy FAT32 we first have to split the `install.wim` because it is larger than FAT32 can handle.
The Split is done by:

`Dism /Split-Image /ImageFile:W:\sources\install.wim /SWMFile:U:\sources\install.swm /FileSize:4000`

After we have split the install file we can continue with all other files but we have to add the large `install.wim` to an exclude list

create a new text file and put `sources\install.wim` in the first line, close it and save (In this sample it's U:\exclude.txt)

Run `xcopy.exe W:\*.* U:\ /E /H /F /EXCLUDE:U:\exclude.txt`

and now the Legacy system will also be able to boot to your stick!
