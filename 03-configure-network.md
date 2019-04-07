# Configure Network

We want our router for:

1.  DHCP giving out _dynamic_ IP addresses to devices connecting to the network.

2.  Assigning a _static_ IP address to our media server.

## Router settings

If your router has these features, this is probably the easiest.

We're going to use the router to reserve an IP address for the media server on our local network. You'll have to adapt these instructions for your particular router.

Navigate to _Setup > Network Settings_.

Before:

![d-link network config1](./images/d-link1.png?raw=true "d-link1")

After:

![d-link network config1](./images/d-link3.png?raw=true "d-link3")

Scroll down to _DHCP reservation_ section. Select the server computer name from the drop down at the right, and copy over the MAC address with the `<<` button. Make note of the IP address, and boom: That's the IP you'll use for any further configuration of the server.

![d-link network config2](./images/d-link.png?raw=true "d-link")

**note: in this example, the media server is connected to the router via ethernet cable. Because of this, it won't need DHCP to assign its IP address anymore. In the future, the server will not show up in the DHCP client list. To remind myself (or leave a record for others) about the rule, I don't delete the DHCP reservation information. Listing reservations as active here prevents the router from handing out the same IP addresses to other clients on the network.**

Save settings and reboot the router.

`ping 192.168.x.xxx` to test connection.

## Domain resolution with `/etc/hosts`

On the dev box, the `/etc/hosts` file is where your computer will resolve domains _locally_ before reaching out to the network for DNS. We're going to create entries for our media server here. Any name we want to resolve to our server's ip address will be assigned here.

Open up the file: `sudo vim /etc/hosts`.

Your entry should be similar to this:

```bash
# media server
192.168.x.xxx your.media.server

# virtualbox server should have its own entry
192.168.y.xxx media.server.testing
```

Assign whatever name you like. This is what you type into the browser instead of the IP address.

Save and close the file.

Now in the browser, type in `your.media.server`(`media.server.testing` for the vbox) or whatever name you've assigned to the server in the hosts file.

This is slightly better than typing in the ip every time, BUT

1.  we still have to manually enter the port number and web path, AND

2.  domains assigned in the hosts file will only resolve _locally_ (on the dev box).

### Up next:

[Install Plex](./04-install-plex.md)
