# Install Virtual Box

Adapted from:

-   <https://github.com/PuddletownDesign/Linux-Setups/blob/master/local-development-server-with-virtualbox.md>

A headless virtual webserver running vanilla Debian with shared folders.

```bash
brew cask install virtualbox
```

Grab a Debian Net installer torrent

<https://cdimage.debian.org/debian-cd/current/amd64/bt-cd/>

From the bottom of this page, it looks like this. Do not grab the one labeled mac, if you're using a mac.

`debian-x.x.x-amd64-netinst.iso.torrent`

after it's complete, ¡Keep the torrent seeding for the community!

Copy the img to a new more permanent location, like `~/Documents/Dev/isos/`

## Create a VM

1.  open virtualbox gui

2.  new machine

3.  name the machine

4.  give it 512 ram

5.  Create a new disk (VDI) - Dynamically allocated

6.  Default 8GB - 20GB depending

7.  Start in normal mode

8.  Select the Debian ISO you downloaded as a torrent debian-9.1.0-amd64-netinst.iso

## Install Debian in virtualbox

Apply installation instructions from [the first section of this guide](./01-install-debian-from-usb.md).

### Set network adapters

In Virtual Box, **preferences -> network -> Host only.**

Create a new "host only" network (make sure if other VMs exist to increment the IP by one). Make sure DHCP server is enabled.

In the machine settings panel go to networks

1.  The first adapter needs to be set "Host only"

    -   SSHing from dev box to the vm via the host only IP

    -   Select the adapter you just made for it in preferences next to "name"

2.  needs to be "enabled" and set to NAT

    -   provides network interface for the guest machine.

Turn the machine back on and log in with your user. (We won't be using root anymore)

Open up the network interfaces file

```bash
sudo nano /etc/network/interfaces
```

Modify it to the following, replace with your IP.

```bash
# This file describes the network interfaces available on your system

# and how to activate them. For more information, see interfaces(5).

source /etc/network/interfaces.d/\*

# The loopback network interface

auto lo
iface lo inet loopback

# Host-only interface

auto eth1
iface eth1 inet static
    address         192.168.xx.xx
    netmask         255.255.255.0
    network         192.168.56.0
    broadcast       192.168.56.255

# NAT interface

auto eth2
iface eth2 inet dhcp
```

now reboot the machine

```bash
sudo reboot
```

## Guest Additions and shared folders

For our virtualbox server (guest), we're going to install and set up Guest Additions, as well as create a shared folder on the host (dev box).

A few reasons:

-   single reference point

    -   deployed on shared hardware

    -   no need for `scp` or `smv`

-   keep the total disk space of the VM down

    -   avoid making duplicates of media files

### Download the correct Guest additions

Install dev tool dependencies

```bash
sudo apt-get install linux-headers-$(uname -r) build-essential dkms -y
```

Find the correct package from this list:

<http://download.virtualbox.org/virtualbox/>

### Install Guest Additions

```bash
cd ~

# replace with your version
wget <http://download.virtualbox.org/virtualbox/x.x.x/VBoxGuestAdditions_x.x.x.iso>

# Make a directory to mount the guest additions
sudo mkdir /media/VBoxGuestAdditions

# Now mount them to above directory
sudo mount -o loop,ro VBoxGuestAdditions_x.x.x.iso /media/VBoxGuestAdditions

# Now install them
sudo sh /media/VBoxGuestAdditions/VBoxLinuxAdditions.run

# Unmount them
sudo umount /media/VBoxGuestAdditions

# Delete the directory you created to hold them
sudo rmdir /media/VBoxGuestAdditions

# Delete the iso unless you want to keep it
rm VBoxGuestAdditions_x.x.x.iso
```

### Configure shared folder

In Virtualbox: **_Settings > Shared Folders_**

_While VM is running_, it should be set to "Make Permanent" and "Auto-Mount". The "Make Permanent" checkbox doesn't show up until the VM is running.

![vbox-settings](./images/vboxShare.png?raw=true "vboxShare")

"Auto-mount" will mount the shared directory in `/media/` at boot, and "Make Permanent" tells the VM to treat the mounted directory as though it were part of the original file system.

```bash
username@servername:~$ ls -l /media/
total 4
lrwxrwxrwx 1 root root      6 Jun 27 18:49 cdrom -> cdrom0
drwxr-xr-x 2 root root   4096 Jun 27 18:49 cdrom0
drwxrwx--- 1 root vboxsf  306 Aug 13 00:11 sf_Media
# The mounted folder here is "Media". Yours may be differently labeled.
```

If the mount was successful, the directory will have `sf_` prepended to it.

### Group permissions

Note the permissions of `sf_Media`:

-   owned by `root` = `7`

-   `rwx` permissions for group `vboxsf` = `7`

-   no permissions for anyone else = `0`

(`drwxrwx---` == `770`)

Check what groups your user belongs to:

```bash
groups username
```

If `vboxsf` isn't listed, add your user to the group:

`sudo usermod -a -G vboxsf username`

## Symlink the Guest Plex Library

By default, Plex serves media from `/var/lib/plexmediaserver` (and corresponding subdirectories).

Since all of the media we want to access is already mounted in our shared folder, we can create a symlink for the Plex media library to read from `sf_Media`.

#### _Troubleshooting: Symlinks_

If there is already a directory called `Media` in the location we're trying to place the link, the command will fail.

This is because symlinks cannot link two _existing_ directories. Think of the symlink as 'mounting' one directory in another location, or "forking" its own file path through another directory.

**_Resolution_**: Replace the conflicting directory with the new symlink.

##### safely:

Backup the original directory and give it a new name (a possibility is to append with `.bkp`)

```bash
mv /var/lib/plexmediaserver/path/to/Media /var/lib/plexmediaserver/path/to/Media.bkp
```

##### destructively:

```bash
rm -rf /var/lib/plexmediaserver/path/to/Media
```

### Create the symlink:

```bash
ln -s /media/sf_Media /var/lib/plexmediaserver/Library/Application\ Support/Plex\ Media\ Server/Media ## may have to use `sudo`
```

We now have Plex reading the symlink `Media` from the mounted shared folder `sf_Media` into the default Plex path (`/var/lib/plexmediaserver/path/to/your/Media`).

Let's create another symlink to the Media directory in `/home/username/`, keeping our access paths short:

```bash
ln -s /media/sf_Media /home/username/Media
```

Now we can just:

```bash
ls -l ~/Media/
```

And the proper directory contents are listed.

### Conclusion

We can add and remove files inside the shared directory from either the guest or the host. You want to be very careful when removing media, because **IT WILL DISAPPEAR IN BOTH PLACES**, so backups are your friend, my friend.
