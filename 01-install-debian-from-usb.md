# Install Debian

## Create Debian installer on USB stick

Get an official .iso image file from a reliable source:

<https://www.debian.org/distrib/netinst#smallcd>

Select the appropriate architecture for the server machine, then download directly or torrent from a mirror.

Insert the blank USB drive. Find out the name of the drive:

```bash
diskutil list
```

The disks of your computer will look similar:

```bash
/dev/disk0 (internal, physical):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:      GUID_partition_scheme                        *750.2 GB   disk0
   1:                        EFI EFI                     209.7 MB   disk0s1
   2:                  Apple_HFS HDD                     749.8 GB   disk0s2

/dev/disk1 (internal, physical):
   #:                       TYPE NAME                    SIZE       IDENTIFIER
   0:      GUID_partition_scheme                        *1.0 TB     disk1
   1:                        EFI EFI                     209.7 MB   disk1s1
   2:                  Apple_HFS SSD                     999.3 GB   disk1s2
   3:                 Apple_Boot Recovery HD             650.0 MB   disk1s3
```

There should be one _below_ that says something similar:

```bash
/dev/disk2 (external, physical):
```

That will show the information of the external drive.

Be **_very_** careful with this command, as `dd` writes over the entire target disk. The `SIZE` of the disk, and the part where it says `external` are some clues to look for.

Now that you have the image, issue the `dd` command to create the bootable USB. You may have to use `sudo`

```bash
dd if=/path/to/image/file.iso of=/path/to/rdisk2 bs=1m
```

Notice the `rdisk2` instead of `disk2`, and the `bs=1m` at the end of the command. These two speed up the `dd` command exponentially. If you don't add either, or if you choose to add just one of them, the command will consume more time. `man dd` for more info. **You can use ctrl + t to check on the progress of dd**

The `dd` command will write to the drive. The `if=/path/to/image/file.iso` tells the command which .iso file to use, and `of=/path/to/rdisk2` tells the command which drive to write to.

**_note_**: There are several _non-command-line_ options for creating bootable live USBs, including:

-   UNetbootin

<https://unetbootin.github.io/>

-   Etcher

<https://etcher.io/>

## Install Debian

Now that we have the installation USB, we'll insert it into the computer that we want to install Debian on. If the computer is on, power it down. Reboot while holding `F2`. This will give us the bios menu. From there, look for the boot menu options. Select the USB device that you've inserted into the computer and move it to the top of the boot order. Save the configurations and reboot. The machine will boot into the Debian installer USB.

_Section adapted from_

<https://github.com/PuddletownDesign/Linux-Setups/blob/master/local-development-server-with-virtualbox.md>

1.  Select `install` not `graphical install`

2.  Run through your language and regional settings

3.  Set your hostname to whatever you want

4.  Hit continue for domain name

5.  Set up a root password

6.  Set up the account name that you will use

7.  Timezone settings

8.  Select guided entire disk for partition

9.  All files in one partition

10. Finish partitioning and write changes to disk `/ Yes`

11. Select `no` do not install from the cd

12. Select a network mirror close to you

13. Skip Http Proxy information unless it applies to you

14. Skip package survey question `no`

15. Select the packages to install

    -   In this part, **USE SPACEBAR** to select items **DO NOT HIT RETURN**

    -   Only install:

        -   `Standard Linux Sysem`

16. Install Grub Bootloader

17. Select `/dev/sda`

18. Boot into the system

    -   `Reboot`

    -   Login with `root` and the root password you created

19. Install `ssh`, and editor, and `net-tools`

```bash
apt-get update
apt-get -y install sudo ssh openssh-server vim net-tools aptitude
systemctl start ssh.service
systemctl enable ssh.service
```

20. Add user to sudo and poweroff:

```bash
usermod -a -G sudo username
poweroff
```

### Up next:

For Development Vbox:

[Virtual Box](./07-configure-for-virtualbox.md)

or if you're not using Vbox:

[Configure SSH](./02-configure-ssh.md)
