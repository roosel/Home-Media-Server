## MOTD (optional)

The `motd` file contains the message of the day. This file is only used for static text; no scripts can be put here. For now, remove all the text.

```bash
sudo vim /etc/motd
```

Add a multicolored fortune telling cow to your ssh login instead:

```bash
sudo apt-get install cowsay fortune -y
```

Replace the contents of the `/etc/update-motd.d/10-uname` file. This file prints out the string of the OS and last login.

```bash
sudo vim /etc/update-motd.d/10-uname
```

Add the following:

```bash
W="\033[01;37m"
B="\033[01;34m"
R="\033[01;31m"
G="\033[01;32m"
X="\033[00;37m"

echo "$G"
uname -snrvm
echo "$B"
/usr/games/fortune | /usr/games/cowthink
printf "\n$R"
```

This is a fun improvement for our logins.

For more customization, check out `man cowsay` or `cowsay -l` for a list of arguments.

Other programs that output OS and login information:

-   `archey`

-   `screenfetch`
