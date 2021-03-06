# Installing FOG Server
FOG will run on a variety of Linux disros, however Ubuntu is probably the most beginner friendly disto to run. As such, this tutorial will provide a step-by-step guide to getting FOG Server up and running on Ubuntu Server 16.04. *Enjoy!*

**Table of Contents:**
- [Create a bootable USB with unetbootin](#bootable-usb)
- [Install Ubuntu](#install-ubuntu)
	- [Partitioning](#partitioning)
		- [SWAP](#swap)
		- [Root](#root)
	- [Software Selection](#software-selection)
	- [Bootloader](#bootloader)
- [Configuring Ubuntu](#configuring-ubuntu)
	- [Logging in](#log-in)
	- [Install Ubuntu Desktop](#ubuntu-desktop)
	- [Installing Essentials](#essentials)
	- [Giving control to Network Manager](#network-manager)
	- [Assigning a static IP address](#static-ip)
	- [Download and install FOG Server](#install-fog)
	- [Configuring DHCP](#dhcp)

<a name="bootable-usb"></a>
## Create a bootable USB with unetbootin

Download [Ubuntu 16.04 Server](http://releases.ubuntu.com/16.04/ubuntu-16.04-server-amd64.iso.torrent). This is a torrent link, so be sure to have Transmission, qBitTorrent or similar installed.

Ensure that the target USB drive is FAT formatted prior to writing the image with unetbootin.

Download [unetbootin](https://unetbootin.github.io/).

Open unetbootin and select the "Diskimage" radio button, then select the ISO file we downloaded. Set "Type" to "USB Drive" and "Drive" to the whatever USB drive you have selected to use.

<a name="install-ubuntu"></a>
## Install Ubuntu

Follow the install instructions for Ubuntu Server, most of the defaults are fine. 

<a name="partitioning"></a>
### Partitioning
When you reach the "Partition Disks" screen select Manual then select the primary Hard Drive from the following menu, agree to create a new partition table.

<a name="swap"></a>
#### SWAP
- Select FREE SPACE
- Create a new partition
- Set the size of this partition to twice that of size of the RAM installed on the system
- Set type to Primary.
- Set Location to Beginning
- Set "Use as" to "swap area", then continue to "Done setting up the partition."

<a name="root"></a>
#### Root 
- Select FREE SPACE again
- This time set the size to "max"
- Set type to Primary.
- Set "Use as:" to "Ext4 [...]"
- Optionally set a disk label
- Turn the bootable flag on, then continue to "Done setting up the partition."

Finally "Finish partitioning and write changes to disk", and accept. Ubuntu will now continue to install the base system, so feel free to grab a cup of coffee and kick back.

When prompted to configure automatic updates, select "No automatic updates."

<a name="software-selection"></a>
### Software Selection
Ubuntu Server has a variety of optional software installs. We'll try to keep it lightweight, for now, by only installing standard system utilities. We'll install Ubuntu Desktop after the installation. Continue.

<a name="bootloader"></a>
### Bootloader
Install GRUB on the MBR of your primary hard drive.

Shortly after, the installation will complete and you can now remove the USB drive and boot the system.


<a name="configuring-ubuntu"></a>
## Configuring Ubuntu

<a name="log-in"></a>
### Logging in
Logging in is simple enough, however while writing this guide I ran into an issue where Ubuntu gave me odd, unhelpful but also non-critical messages about the disk upon start-up. I had to open tty2 by pressing `Ctrl`+`Alt`+`F2`, at this point I could log in properly with the credentials I created in the installation step.

<a name="ubuntu-desktop"></a>
### Install Ubuntu Desktop
The reason I chose not to install Ubuntu Desktop initially is because Ubuntu ships with quite a few unneeded packages that bloat the installation. To avoid this, we can install it after the initial installation with some additional options.

- First update apt-get by running `sudo apt-get update`
- Then install Ubuntu Desktop by running `sudo apt-get install ubuntu-desktop --no-install-recommends`

This can take a while so maybe get some more coffee?

After Ubuntu Desktop finishes installing, reboot the system by running `sudo reboot`. After the system reboots you should be greeted by Ubuntu's Unity Interface.

<a name="essentials"></a>
### Installing Essentials
You'll notice ubuntu looks a little bare, without even any way to logout or open a terminal. This is all thanks to `--no-install-recommends`. We'll now install the bare essentials for our server. Since we have no terminal, we'll switch over to tty1 by pressing `Ctrl`+`Alt`+`F1`. Log in and install the following packages with the format `sudo apt-get install [packages]`, you can install as many space-seperated packages at one time as you would like.

- System utilities (required): `software-center indicator-applet-complete gnome-terminal gnome-disk-utility gnome-network-manager git`

- Lens components (required): `unity-lens-applications unity-lens-files`

- SAMBA (recommended for print driver network share): `samba nautilus-share`

- Apps (optional/preference): `firefox transmission`

- Resource Monitor (awesome): `indicator-multiload`

After you've finished installing all the packages that you need, go ahead and reboot with `sudo reboot`.

<a name="network-manager"></a>
### Giving control to Network Manager
The Network Manager applet should now show up in menu bar at the top of the screen, however, your network devices will show up as being unmanaged. To allow Network Manager to manage our devices, open the Terminal (which you can now do by searching "Terminal" in lens), and enter `sudo nano /etc/network/interfaces` which will open the configuration file for our network interfaces. Comment all interfaces out using `#`, and exit saving changes. Reboot again, and Network Manager should now have control of our network interfaces.

<a name="static-ip"></a>
### Assigning a static IP address
Click the Network Manager icon in the tray, then select "Edit Connections..." Select the interface you want to assign a static IP address to and click Edit, in my case it was very vaguely named "Wired connection 1". You can now optionally rename the interface. Open the IPv4 Settings, then add an address and configure to your needs. After saving your changes, exit Network Manager. 

**IMPORTANT:** From the Network Manager tray, disconnect and reconnect to you network interface to apply changes. If you do not do this, the FOG installer will configure the server using the old DHCP assigned IP address.

<a name="install-fog"></a>
### Download and install FOG Server
Now it's time to install FOG Server. You can download FOG Server directly from https://fogproject.org/ for a stable release, however I suggest downloading the latest cutting-edge revision from github. The primary features of FOG are always stable, and there are more bug fixes and improvements than there are broken features. At the time of writing, the SHA of the revision used was `88d8bca2b47f1e2446c5a560b37c0fafa8c914dd`. Follow the following steps to download and begin the FOG installation:

- Initial Checkout
```sh
sudo -i
git clone https://github.com/FOGProject/fogproject.git /root/fogproject
```

- Then navigate to the folder you specified and run:
```sh
cd /root/fogproject/bin
./installfog.sh
```

**NOTE:** By default, FOG Server uses PHP 5.3. As of Ubuntu 16, PHP 5 has been removed from the official repos. As of 6/30/2016, the FOG installer needs to be told to use PHP 7 explicitly. Please use the following command when running the installer: `php_ver='7.0' php_verAdds='-7.0' ./installfog.sh`

The FOG installer should now run without any problems. Except for "Would you like DHCP to handle DNS?" which should be N, accept all defaults to the prompts by pressing the `Enter` key.

Later in the install, the installer will prompt you to install/update your database schema. Open the FOG management interface (http://localhost/fog/management) in your web browser, if you're like me you'll be greeted with a message something like this:
```
Your database connection appears to be invalid. FOG is unable to communicate with the database. There are many reasons why this could be the case. Please check your credentials in /var/www/html/fog/lib/fog/config.class.php. Also confirm that the database is indeed running. If credentials are correct, and the Database service is running, check to ensure your filesystem has enough space.
```

This is because MySQL's behavior when the root password is empty has changed so that a non-root user can't log in as the MySQL root user with an empty password. [See here](https://wiki.ubuntu.com/XenialXerus/ReleaseNotes#MySQL_5.7) to learn more about this. To fix this, open a new Terminal instance (leaving the installer running) and log in via the MySQL command line tool as root, finally, change the connection setting.
```sh
mysql -u root
...
mysql> ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '';
mysql> quit
```

If successful, you should now be able to refresh the management interface update the database schema. Return to the installer and hit `Enter`, after a moment the installer should complete. At this point you should be able to return to the management interface and log in with the default credentials: `fog` and `password`.

<a name="dhcp"></a>
### Configuring DHCP
When a client boots to the network, it makes a request to the DHCP server for the necissary information to PXE boot. There are two options for configuring DHCP. Easiest is just to configure the two PXE related options directly on your DHCP server. In my case, the DHCP server was running on a firewall, and didn't allow this level of configuration. In this case my best option was to use proxyDHCP. ProxyDHCP listens and responds to DHCP requests from clients identifying themselves as PXE clients, while leaving the role of assigning IP addresses to the other DHCP servers. I will describe both processes below.

#### Direct Configuration
In most cases, DHCP can be configured for PXE boot by setting two options on your DHCP server.

##### 066 - Boot Server Host Name
This is the IP address or host name of the PXE server, or FOG server, on our network. This will be whatever static IP address you assigned above.

##### 067 - Bootfile Name
This is the name of the PXE image that will be booted from the server. In all modern versions of FOG server since 0.32, this file will be called `undionly.kpxe`.

#### ProxyDHCP
In the case that the DHCP server is unmodifiable for any reason, or you would just like not to modify the DHCP server, proxyDHCP can be used.

1. Install dnsmasq using:
```sh
sudo apt-get install dnsmasq
```

2. Create or modify `/etc/dnsmasq/ltsp.conf` and configure as needed:
```
# Don't function as a DNS server:
# port=0

# Log lots of extra information about DHCP transactions.
log-dhcp

# Dnsmasq can also function as a TFTP server. You may uninstall
# tftpd-hpa if you like, and uncomment the next line:
# enable-tftp

# Set the root directory for files available via FTP.
tftp-root=/tftpboot

# The boot filename, Server name, PXE Server IP Address
dhcp-boot=undionly.kpxe,,x.x.x.x

# !!! Insert the PXE server IP address above !!!

# rootpath option, for NFS
#dhcp-option=17,/images

# kill multicast
#dhcp-option=vendor:PXEClient,6,2b

# Disable re-use of the DHCP servername and filename fields as extra
# option space. That's to avoid confusing some old or broken DHCP clients.
dhcp-no-override

# PXE menu.  The first part is the text displayed to the user.  The second is the timeout, in seconds.
pxe-prompt="Press F8 for boot menu", 3

# The known types are x86PC, PC98, IA64_EFI, Alpha, Arc_x86,
# Intel_Lean_Client, IA32_EFI, BC_EFI, Xscale_EFI and X86-64_EFI
# This option is first and will be the default if there is no input from the user.
pxe-service=X86PC, "Boot from network", undionly

# A boot service type of 0 is special, and will abort the
# net boot procedure and continue booting from local media.
#pxe-service=X86PC, "Boot from local hard disk", 0

# If an integer boot service type, rather than a basename is given, then the
# PXE client will search for a suitable boot service for that type on the
# network. This search may be done by multicast or broadcast, or direct to a
# server if its IP address is provided.
# pxe-service=x86PC, "Install windows from RIS server", 1

# This range(s) is for the public interface, where dnsmasq functions
# as a proxy DHCP server providing boot information but no IP leases.
# Any ip in the subnet will do, so you may just put your server NIC ip here.
# Since dnsmasq is not providing true DHCP services, you do not want it
# handing out IP addresses.  Just put your servers IP address for the interface
# that is connected to the network on which the FOG clients exist.
# If this setting is incorrect, the dnsmasq may not start, rendering
# your proxyDHCP ineffective.
dhcp-range=192.168.200.1,proxy,255.255.0.0

# !!! Insert the DHCP server IP address, and subnet mask above !!!

# This range(s) is for the private network on 2-NIC servers,
# where dnsmasq functions as a normal DHCP server, providing IP leases.
# dhcp-range=192.168.0.20,192.168.0.250,8h

# For static client IPs, and only for the private subnets,
# you may put entries like this:
# dhcp-host=00:20:e0:3b:13:af,10.160.31.111,client111,infinite

```

3. Restart dnsmasq with
```sh
sudo service dnsmasq restart
```

**NOTE:** After this, when I attempted to boot from the network the machine immediately found the DHCP proxy and received the necissary information to PXE boot, however after initializing and establishing a link, I was greeted with this error:
```
TFTP.
PXE-M0F: Exiting Intel Boot Agent. 
```
Initially, I suspected an issue with my dnsmasq configuration, however after inspecting TFTP packets being transmitted over the network to and from this machine using [Wireshark](https://www.wireshark.org/), I found a TFTP packet from the FOG server with `Error code: File not found (1)` for `undionly.0`. For whatever reason, TFTP is still looking for `undionly.0` instead of `undionly.kpxe`. To fix this, I ran the following commands:
```sh
cd /tftpboot
sudo ln -s undionly.kpxe undionly.0
```

**NOTE:** After you have confirmed everything is working, you can change the timeout to 0 on the line:
```
pxe-prompt="Press F8 for boot menu", 3
```

### That's all she wrote!
At this point you should have a working FOG server. You may still have to tinker with some of FOG's settings, particularly the *FOG Configuration>iPXE Boot Menu settings>Exit to Hard Drive Type* setting, these will depend on several factors including BIOS of your target machine. Otherwise, enjoy your now copious spare time!