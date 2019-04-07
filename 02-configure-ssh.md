# Configure SSH for remote access

Adapted from:

<https://github.com/PuddletownDesign/Linux-Setups/blob/master/local-development-server-with-virtualbox.md>

## Install SSH

Now that we're signed in with root, let's install SSH. We'll install `vim` along with it for easy editing (you can use `nano` if you prefer).

first, update:

```bash
apt-get update -y

# Then install `ssh`, `vim` (or `nano`) and `net-tools`

apt-get -y install ssh openssh-server vim net-tools sudo aptitude
```

Start and enable ssh service

```bash
systemctl start ssh.service

systemctl enable ssh.service
```

Now grant `sudo` privileges to your normal user account by adding it to the `sudo` group. Make sure to adjust the following command for your username.

```bash
usermod -a -G sudo username
```

Now your user account has sudo privileges! When changing permissions and groups for users, you need to logout and log back in for those changes to apply. Let's do exactly that. For now, we won't need to use the root account.

After logging in, `sudo apt-get update -y` should return no errors.

Open up the network interfaces file:

```bash
sudo vim /etc/network/interfaces
```

The file should read similar to the following:

```bash
# This file describes the network interfaces available on your system
# and how to activate them. For more information, see interfaces(5).

source /etc/network/interfaces.d/*

# The loopback network interface
auto lo
iface lo inet loopback

# The primary network interface
allow-hotplug enp63s0
iface enp63s0 inet dhcp
```

If you had to make any changes, save and close the file. Reboot the machine.

```bash
sudo reboot
```

### Login via SSH

On your main computer (mac), login to the server with ssh and the new user information.

```bash
ssh username@192.168.*.***
```

You'll be prompted for the user password that you created during installation.

By default, Debian will not allow root login with password; you'll need to set SSH public keys.

You will need to login using the user account that we created during install. We will need to edit the `sshd_config` file.

### Configure SSH for convenience & security

We want to configure SSH for three things:

1.  Not allow `root` login via `ssh` (lock it down)

2.  Require authentication from SSH keys and disallow password logins for all users.

3.  Login via `ssh` from another computer on the network with an alias configured in the `~/.ssh/config` file of that machine.

Now that you're logged in with your user and have `sudo` privs, we're going to lock SSH down.

#### Set up key pairs for passwordless login

In this step we will be assigning your main computer public keys to log you into the Debian machine instead of using passwords. Keys are much more secure and (BONUS!) don't require typing passwords.

#### Generate keypairs:

If you don't already have ssh keys on your dev box (mac), then follow this to generate them. Otherwise skip this step.

```bash
ssh-keygen
```

Then save them to the default file.

```bash
/Users/localuser/.ssh/id_rsa
```

Next, copy the SSH keypairs to the media server OS (Debian)

```bash
brew install ssh-copy-id

ssh-copy-id username@ipaddress
```

Enter your password for the account.

Now try logging in!

```bash
ssh username@ipaddress
```

If everything is working well you should login with no password!

### Create an ssh alias for simpler login

On the main computer, open up the ssh configuration file.

```bash
vim ./.ssh/config
```

Add your media server similarly to this at the bottom of the file:

```bash
Host nameOfServer
	RemoteForward 52698 localhost:52698
	HostName 192.168.*.***
	User username
	Port 22
```

Save and close the file.

On the server, lets edit the `sshd_config` file. _NOT the `ssh_config` file_

```bash
sudo vim /etc/ssh/sshd_config
```

Change the following line first

`#PermitRootLogin prohibit-password`

to

`PermitRootLogin no`

Make sure to uncomment the line as well. If you are having a hard time finding it, its located under the authentication header.

Next change this line:

`#PasswordAuthentication yes`

to

`PasswordAuthentication no`

Now restart the SSH service

```bash
sudo systemctl restart ssh
```

Before we log out of the server, we should test our new configuration. We do not want to disconnect until we can confirm that new connections can establish successfully.

Open another terminal tab or window. In the new tab or window, we need to make a new connection to our server. Again, instead of using the root account, we want to use the new account that we created, and we want to use the spiffy alias we added to our local `~/.ssh/config` file.

Login with the alias. Adjust the line below to fit your alias.

```bash
ssh yourAlias
```

If it is not working, go back and recheck the server `sshd_config` file, or double-check dev box's  `~/.ssh/config` for syntax issues.

Now we are securely logged in, using no password, with our mega-cool alias!

### Up Next:

[Configure Network](./03-configure-network.md)
