# The world of the Pi Hole

## A comprehensive setup guide

### Precursor

Pihole is a nice bit of software which accomplishes several things for you.

1. Becomes your new DNS server
2. Blocks any unwanted domains
3. Whitelist domains from those unwanted ones which you need
4. Web interface to deny / approve / edit on the fly from any device on your network
5. Option for a DHCP server
6. Option to set up as a VPN from anywhere

While it does all of these things, it's not a perfect solution. This guide uses a public list for the blacklist of hosts found here: https://github.com/StevenBlack/hosts/blob/master/hosts

This blacklist is update often, and this guide will include a script to pull a new version and update your blacklist file any time you want (can be run from an SSH session for ease-of-use)

But, this blacklist will not block everything you want. Or it may. Depends on what you want out of this. I have over 600k blocked and I'm getting to a good point in my eyes, there's lists for everything you may want to block online including regex (which can be put in via the CLI or website)

----------------------

### Prerequisites

1. A device capable of running [Pihole](https://pi-hole.net/)
 a. This can be pretty much any OS if you want to use a container. This guide will cover the automated installer on a Raspberry Pi
 b. Boons of using a container are obvious but useful if you dont want a separate device dedicated (or even if you do). Always runs on startup, doesnt go down, etc

.. Thats it

### Setting up your Raspberry Pi

1. Pick a suitable model for your needs. I went with a 3B+ because it has an ethernet port, and I don't need much more power for this.
2. Download the Raspberry Pi OS installer
3. Use a flash drive or SD card as the installation medium for the OS
4. Install and make sure it boots. I'd recommend not using special characters in your password, as the default keyboard layout is horrific and it's easy to assume you set the password incorrectly.

------------------------

### Setting up PiHole

#### Set up a static IP for your Raspberry Pi

1. Grab your current ipv4 address for the rpi using `hostanme -I`
2. Grab your router's ip address using `ip r`
3. Get the IP address of your DNS using `grep "nameserver" /etc/resolv.conf`
4. Edit the dhcp config. It can be in a different spot depending on your OS version/OS choice, generally it's in `/etc/dhcpcd.conf` or `/etc/dhcp/dhclient.conf`. Include the following text at the end of the file:

```conf
interface [INTERFACE; Such as en0 or wlan0]
static_routers=[IP OF ROUTER]
static domain_name_servers=[IP OF DNS]
static ip_address=[STATIC IP ADDRESS YOU WANT FOR THE RPI]/24
```

#### Download and install Pi Hole

1. Run `wget -O basic-install.sh https://install.pi-hole.net; sudo bash basic-install.sh` to download & run the install script
2. Run through the install script, making choices of your desires

#### Set up blacklist

##### The script is optional, you can add links to files on the web that contain lists on the UI, and run `sudo pihole -g` to populate the DB. This script makes sure the address is reachable by your router, before it adds, to not populate with useless addresses (duplicates, out-of-your-country, already blocked by your ISP, etc)

1. Grab this script to populate your blacklist from a list of hosts: `wget https://gist.githubusercontent.com/hunterkepley/c0c54308b3d111d4ca3528a2b612e244/raw/64b85a60d9c0ee85dfcfd798431ab506cc76c704/sh`
2. Grab list of hosts (of your choosing, I am using [this one](https://raw.githubusercontent.com/StevenBlack/hosts/master/hosts). using `wget` to make it easy
3. Make sure you're in `~/`
4. Run `mkdir src`, `mkdir src/out`, `touch src/out/combined.raw`
5. If you need to run `pihole -g` using sudo, edit that portion of the script to include it. Doesn't matter, you can run after the script is done anyway
6. `chmod +x` the script to make it executable
7. Run the script
8. Profit

###### *Important*: This blacklist list is ENORMOUS. The script pings hosts it finds, and if your router can access them, it adds them to your blacklist (does not replace the blacklist). This will take forever to run, on a Raspberry pi, since it's slow + a slow process. You may want to run this on a faster computer, then put the blacklist output on your rpi. 

#### Logging into the web interface

Pihole should give you a link (it's the static IP of your rpi with `/admin` added on) and a default password to log in with. You can use this to look at blocked queries, manage Pihole without SSHing into your rpi, etc.

I did the whole set up without GUI, it's pretty simple overall

#### Maintaing and adding onto your blocklist/allowlist

Once you do the initial setup, feel free to look around for different blocklists. Grabbing the raw file on github, you can easily feed it into the Lists on the web UI and run `sudo pihole -g` on your device to add them to the database.

I am going through (this repo)[https://github.com/hagezi/dns-blocklists] for example, and grabbing lists which make sense to me


One thing you can do, is run the script on a copy of your blacklist. `pihole -g` will tell you how many duplicates you have, and running the script is an easy way to clean it up. It will take time, but you can run-it-and-forget-it.

To do so, make a copy of your blacklist, clear the original, and run it using the copied blacklist. This way it doesnt combine with your existing blacklist

#### DNS settings on router

Ideally, you want only pi hole as your DNS server. If this is not working well due to load/time down/etc, you can set up a second pi hole device (on another rpi, or wherever) as a secondary DNS. Another option is using an ad-blocking DNS if you want at *least* that functionality.

If you have `8.8.8.8` or another normal DNS as your secondary DNS, you will still receive some blocked traffic

#### About Logging and bad performance

Logging queries takes up a LOT of resources on an rpi. I recommend turning it off once youre in a good spot and are happy with what is blocked/what is not. You can always re-enable this later. In my personal setup, I have one RPi doing logging + the main workload, and a Pi Zero for doing the work it can't with logging disabled.

You can still see queries, it just won't save logs from what I can tell.

At the end of the day, it's all about what fits your setup. I'm going to use a 3rd and 4th eventually for other projects, and may use some of the resources on the Zero for communication between those 2.
