# Plex

The program for serving our media.

## Download:

Login to the server (with password-less ssh!). Paste the following lines (check for latest version, and adjust the `wget` address accordingly.

```bash
cd /tmp

wget https://downloads.plex.tv/plex-media-server/1.8.4.4249-3497d6779/plexmediaserver_1.8.4.4249-3497d6779_amd64.deb
```

## Install:

```bash
sudo dpkg -i plexmediaserver_1.8.4.4249-3497d6779_amd64.deb
```

Should return something similar to this:

```bash
Selecting previously unselected package plexmediaserver.
(Reading database ... 81263 files and directories currently installed.)
Preparing to unpack plexmediaserver_1.8.4.4249-3497d6779_amd64.deb ...
Unpacking plexmediaserver (1.8.4.4249-3497d6779) ...
Setting up plexmediaserver (1.8.4.4249-3497d6779) ...
Created symlink /etc/systemd/system/multi-user.target.wants/plexmediaserver.service → /lib/systemd/system/plexmediaserver.service.
Processing triggers for systemd (232-25+deb9u1) ...
Processing triggers for mime-support (3.60) ...
```

### Check server status:

`sudo service plexmediaserver status`

Should return similar to this:

```bash
● plexmediaserver.service - Plex Media Server for Linux
   Loaded: loaded (/lib/systemd/system/plexmediaserver.service; enabled; vendor preset: enabled)
   Active: active (running) since Thu 2017-09-14 14:00:21 WITA; 45s ago
 Main PID: 724 (sh)
   CGroup: /system.slice/plexmediaserver.service
           ├─ 724 /bin/sh -c LD_LIBRARY_PATH=/usr/lib/plexmediaserver "/usr/lib/plexmediaserver/Plex Media Server"
           ├─ 726 /usr/lib/plexmediaserver/Plex Media Server
           ├─ 765 Plex Plug-in [com.plexapp.system] /usr/lib/plexmediaserver/Resources/Plug-ins-3497d6779/Framework.bu
           ├─ 815 /usr/lib/plexmediaserver/Plex Tuner Service /usr/lib/plexmediaserver/Resources/Tuner/Private /usr/li
           ├─ 822 /usr/lib/plexmediaserver/Plex DLNA Server
           ├─ 847 Plex Plug-in [com.plexapp.agents.plexmusic] /usr/lib/plexmediaserver/Resources/Plug-ins-3497d6779/Fr
           ├─ 966 Plex Plug-in [com.plexapp.agents.thetvdb] /usr/lib/plexmediaserver/Resources/Plug-ins-3497d6779/Fram
           ├─1055 Plex Plug-in [com.plexapp.agents.plexthememusic] /usr/lib/plexmediaserver/Resources/Plug-ins-3497d67
           ├─1064 Plex Plug-in [com.plexapp.agents.opensubtitles] /usr/lib/plexmediaserver/Resources/Plug-ins-3497d677
           ├─1130 Plex Plug-in [com.plexapp.agents.themoviedb] /usr/lib/plexmediaserver/Resources/Plug-ins-3497d6779/F
           └─1242 Plex Plug-in [com.plexapp.agents.imdb] /usr/lib/plexmediaserver/Resources/Plug-ins-3497d6779/Framewo
```

Access the server from a web browser. Modify the IP address or domain name to match your server. `:32400` is the default port number for Plex, and `/web` is the Plex UI front page.

```bash
# IP
http://192.168.xx.xxx:32400/web

# or domain from /etc/hosts
http://your.media.server:32400/web

# or other domain from /etc/hosts
http://media.server.testing:32400/web
```

Sign up and in with your Plex pass. Create a new one; it's free.

### Copy files over ssh with `scp`

```bash
# from client to server:
scp -r /path/to/file/or/directory/to/be/copied  sshAlias:~/Media/
# the optional `-r` flag tells `scp` that the item to be copied is a directory

# from server to client, invert the command:
scp -r sshAlias:~/Media/desiredMedia /path/to/desired/local/directory/location
```

### Allow LAN clients access to Plex without credentials

We don't want to require clients to sign into Plex on every machine each time Plex is opened. Cuz that's annoying.

To achieve this, we'll specify a range of IP addresses that can access our Plex server "without authentication".

It works like this:

1.  device connects to the wireless network

2.  device is assigned an IP via DHCP server

3.  if assigned address is within the specified range, then:

4.  allow device access to server without credentials

![plex-settings](./images/plex3.png?raw=true "plex3")

Navigate to the settings tab, then the server tab, then the network tab on the left.

![plex-settings](./images/plex2.png?raw=true "plex2")

Scroll down "List of IP addresses and networks that are allowed without authentication" section. Enter the IP range for your local network:

![plex-settings](./images/plex1.png?raw=true "plex1")

```bash
192.168.x.0/24 # or whatever your IP range is
```

Save settings.

Now in the browser on another device on the network, you can type in the IP address with the port `:32400` and `/web`, accessing the server without having to enter any credentials.

```bash
192.168.x.xxx:32400/web
```

Now we can access our server from other devices on the network. We still have to manually type out the IP address, port, and path, though.

To simplify web UI access, we _could_ modify the hosts file for each client when they connect. If this is a small enough network with only a few clients, sure. To present services to clients on our network with an easy to remember domain, we'll need a slightly more elegant solution.

Now we can add media libraries to our server through the web UI.

### Organizing media

Plex is picky. If it doesn't see a naming convention it recognizes, that media will be ignored.

The formatting is fairly straightforward.

For example, television can be labeled `show title - 01x01 - episode title`. For Christmas and other specials, `00x01` can be used.

```bash
Media
	Television
		Show
			Season 1
				"show title - 01x01 - episode title"
			Specials
				"show title - 00x01 - episode title"
	Movies
		Movie Title (year)
			"movie title - year"
		Movie Title 2 (year)
			"movie title 2 - year"
```

It's possible to resolve a lot of media recognition issues through reorganization. Leave the server running and keep an eye on the web UI; when you organize the files in a way that Plex recognizes, it will automatically populate your library with the corresponding metadata.

If you haven't already, after organizing and labeling media, securely copy files over:

```bash
scp -r /path/to/local/media/directory serverAlias:/path/to/server/location/media/directory
```

Once finished, we'll open up our browser and navigate to our server again:

```bash
# created in the hosts file:
your.media.server:32400/web
```

![plex-settings](./images/plex5.png?raw=true "plex5")

Click on the `+` sign to the right of "LIBRARIES"

![plex-settings](./images/plex6.png?raw=true "plex6")

Select the type of library you'll be adding and give it a title

![plex-settings](./images/plex7.png?raw=true "plex7")

Point Plex at the directory with stored media

![plex-settings](./images/plex8.png?raw=true "plex8")

#### _Troubleshooting: Media not recognized when importing library_

You may run into an issue where some of your media isn't recognized (doesn't appear in the web UI) when you create a library. 9 times out of 10, this is due to the aforementioned naming convention. Try renaming and organizing your library before digging too deep.

##### Example:

Let's say you have a TV show with several seasons and a movie in a directory, each in their own subdirectories. Example:

```bash
Television
	TVshow
		Season 1
			01x01 Pilot
			01x02 Etc.
		Season 2
			02x01 atItAgain
		TVshow Movie
			Name of TVshow, movie
			Name of TVshow movie 2, electric boogaloo
		Holiday Specials
			00x01 holidayEpisode
			00x02 holidayEpisode2
	TVshow2
		Season 1
			01x01 Pilot
			...
Movies
	Movie (year)
		Move (year).ext
```

When you point the Television Library at the `Television` folder, the `TVshow` and `Seasons` subdirectories are recognized by Plex, but the `TVshow Movie` subdirectory is not. It's obviously not a TV show, but attempting to `mv` or `cp` the individual `TVshow Movie` subdirectory to the `Movies` folder (Where you have the Movies Library aimed) doesn't recognize it, either.

To make it visible to Plex, we're going to add another entry to the "Movies" Library, pointing it at the `TVshow Movie` subdirectory in the `TVshow` folder. This points to the directory, adds it to the Library, then Plex takes care of that metadata martial arts. See figure below:

![plex-settings](./images/plex4.png?raw=true "plex4")

_I've added the movie from within the Television directory_

_Notice "Strangers.with.Candy.2005" Movie added from the TV show directory in there?:_

![plex-settings](./images/plex9.png?raw=true "plex9")

_We added the "Strangers with Candy" TV show directory to our Movies Library, and Plex recognized the "Strangers with Candy" Movie contained within that directory while ignoring the tv shows. It then posted to the Movies Library in the web UI and started grabbing metadata:_

![plex-settings](./images/plex10.png?raw=true "plex10")

This keeps my existing directories and subdirectories intact. Rather than moving files around on the system, aiming the UI at multiple folders allows me to avoid scattered content and disorganized file structure on the backend, while keeping Plex happy and things looking nice on the frontend.

### Up next:

[Configure for Virtualbox](./05-configure-for-virtualbox.md)
