{{Warning|If you're thinking of mounting /usr/portage, /var/tmp, or, (heaven forbid!) swap across a network, you're thinking potentially dangerous thoughts. Take into account that networking is only half as reliable as "regular" flash or disk storage before attempting those maneuvers. (And, no, that's not a precise statistic.)}}

We define "host" as the machine with the Internet access, and "client" as the machine needing Internet.

Here we describe how to set up a networking connection between two machines via USB. We've broken this page up into two parts, depending on what your goals are:
* Simply plugging in the device and having the host share its Internet with the client is the easiest option, but it means that the client never sees the network the host is on. If you want just a simple client<->host setup, see [[#Simple Networking]]
* Having the client's network join the host network requires a network bridge. This is the topic of the section [[#Bridging]].

== Simple Networking ==

=== Prerequisites ===
* Linux. (Windows is possible, but these instructions only apply to Linux)
* A mini-B usb cable.
* Patience. Unfortunately, NAT in Linux can be extremely difficult to figure out, even after following instructions like these.

{{Warning|NetworkManager, which comes pre-installed on most GNOME-based desktop Linux distributions, may mess you up if you try to manually configure the usb0 interface.}}
{{Warning|The same applies to netplug. Some variants have netplug pre-installed and will automatically launch or close dhcpcd depending on whether or not the USB cable is plugged in. This will mess you up if you have it on while trying to manually configure usb0.}}

=== Host Setup ===
All the following instructions apply to the host machine only.

==== Compile Kernel with IP Masquerading Support ====

{{Kernel|2.6.x|
  Networking Support
    Networking Options
      Network packet filtering framework (Netfilter)
        IP: Netfilter Configuration
          <*> IPv4 connection tracking support (required for NAT)
          <*> IP tables support (required for filtering/masq/NAT)
          <*>   Full NAT
          <*>     MASQUERADE target support
  Device Drivers
    [*] Network device support
      USB Network Adapters
    {M} Multi-purpose USB Networking Framework
    -M-   CDC Ethernet support (smart devices such as cable modems)
    [*] USB support
      <*>   Support for Host-side USB
        <*>     EHCI HCD (USB 2.0) support
}}

{{Note|IP networking in the Linux kernel seems to change names and locations as much as Lindsey Lohan goes through *friends, so this may change a bit depending on which kernel you are using. The important part is to get the "usbnet" module compiled, and NAT enabled.}}

==== NetworkManager ====
On the host, if you use NetworkManager with the connection-sharing USE flag, or if you use NetworkManager on Ubuntu, you can '''skip the rest of this page''' after following the below instructions. This is because NetworkManager can handle iptables, DHCP, and DNS setup for you.

First, plug the client in. In the host NetworkManager's Network Connections window you will find "Auto usb0" or similar appear. Edit that connection, select the "IPv4 Settings" tab, and change Method to "Shared to other computers".

Once you apply these settings, NetworkManager will give you an error, claiming there was "no response" from dbus, but it really did work. This is a bug. You will also find that NetworkManager reset all your network connections. This is expected (if not normal).

If you completed these instructions before the client's dhcpcd finished its attempts to get an IP address, NetworkManager will eventually tell you that "Auto usb0", or whatever the connection was named, is now connected. (If you have to start dhcpcd manually on the client, do so.)

You will have to perform these steps every time the device is restarted, because the g_ether kernel module on the client picks a new MAC address every time it is loaded, unless a specific MAC address is hard-coded in.

If this worked, you are all set and no longer need to keep reading.

==== Configuring NAT ====
You configured NAT support in the kernel, but now you need the software to actually take advantage of that NAT support and put it to good use. We'll use the infamous iptables for this.

{{Command|emerge iptables}}
{{Command|/etc/init.d/iptables start}}
(If you add iptables to your default runlevel, you'll never have to start iptables up again by hand. Otherwise, use the above start command before attempting to plug in your client, even after you're done here.)

{{Note|The following was shamelessly copied from the [http://www.gentoo.org/doc/en/home-router-howto.xml#doc_chap5_sect3 Gentoo Home Router Networking Guide].}}

Flush any iptables rules that exist currently.
{{Command|<nowiki>iptables -F</nowiki>}}
{{Command|<nowiki>iptables -t nat -F</nowiki>}}

Setup default policies to handle unmatched traffic
{{Command|<nowiki>iptables -P INPUT ACCEPT</nowiki>}}
{{Command|<nowiki>iptables -P OUTPUT ACCEPT</nowiki>}}
{{Command|<nowiki>iptables -P FORWARD DROP</nowiki>}}

Copy and paste these examples, but change "eth1" to the name of the interface that connects you to the internet, and "eth0" to the interface that appears on the host when the client is plugged in. (Most likely LAN should be set to "usb0".)
{{Command|<nowiki>export LAN=eth0</nowiki>}}
{{Command|<nowiki>export WAN=eth1</nowiki>}}

Then we lock our services so they only work from the LAN
{{Command|<nowiki>iptables -I INPUT 1 -i ${LAN} -j ACCEPT</nowiki>}}
{{Command|<nowiki>iptables -I INPUT 1 -i lo -j ACCEPT</nowiki>}}
{{Command|<nowiki>iptables -A INPUT -p UDP --dport bootps -i ! ${LAN} -j REJECT</nowiki>}}
{{Command|<nowiki>iptables -A INPUT -p UDP --dport domain -i ! ${LAN} -j REJECT</nowiki>}}

(Optional) Allow access to our ssh server from the WAN
{{Command|<nowiki>iptables -A INPUT -p TCP --dport ssh -i ${WAN} -j ACCEPT</nowiki>}}

Drop TCP / UDP packets to privileged ports
{{Command|<nowiki>iptables -A INPUT -p TCP -i ! ${LAN} -d 0/0 --dport 0:1023 -j DROP</nowiki>}}
{{Command|<nowiki>iptables -A INPUT -p UDP -i ! ${LAN} -d 0/0 --dport 0:1023 -j DROP</nowiki>}}

Finally we add the rules for NAT
{{Command|<nowiki>iptables -I FORWARD -i ${LAN} -d 192.168.0.0/255.255.0.0 -j DROP</nowiki>}}
{{Command|<nowiki>iptables -A FORWARD -i ${LAN} -s 192.168.0.0/255.255.0.0 -j ACCEPT</nowiki>}}
{{Command|<nowiki>iptables -A FORWARD -i ${WAN} -d 192.168.0.0/255.255.0.0 -j ACCEPT</nowiki>}}
{{Command|<nowiki>iptables -t nat -A POSTROUTING -o ${WAN} -j MASQUERADE</nowiki>}}

Save permanently so these rules survive a reboot.
{{Command|<nowiki>/etc/init.d/iptables save</nowiki>}}

Tell the kernel that IP forwarding is OK.
{{Command|<nowiki>echo 1 > /proc/sys/net/ipv4/ip_forward</nowiki>}}
{{Command|<nowiki>for f in /proc/sys/net/ipv4/conf/*/rp_filter ; do echo 1 > $f ; done</nowiki>}}

(Optional) To tell the kernel that IP forwarding is OK every time you boot, edit the /etc/sysctl.conf file, and add/uncomment the following lines:
{{File|/etc/sysctl.conf|<pre>
 net.ipv4.ip_forward = 1
 net.ipv4.conf.default.rp_filter = 1
 
 #If you have a dynamic internet address, you probably want to enable this too:
 net.ipv4.ip_dynaddr = 1
</pre>}}

==== DHCP and DNS services ====
The host machine must provide DHCP and DNS services for the client to get online. dnsmasq provides both at the same time, and is very easy to configure.

{{Command|emerge dnsmasq}}
Change usb0 to the name of the interface that appears when the client is plugged into the host, and change the prefix "192.168.0." if so desired.
{{File|/etc/dnsmasq|<pre>
 dhcp-range=192.168.0.100,192.168.0.250,72h
 #Restrict dnsmasq to just the LAN interface
 interface=usb0
</pre>}}
{{Command|/etc/init.d/dnsmasq start}}
(Again, as with iptables, if you don't add dnsmasq to your default runlevel, you will have to run the above start command whenever you want to plug in your client, even after you are done here.)

=== Connecting ===
{{Warning|Do NOT plug in the USB cable yet.  Do the following IN ORDER.}}
{{Note|It's important that iptables and dnsmasq have both been started at this point. Please verify this before you continue if you have not added either to your default runlevel.}}

On the client machine, make sure the g_ether module is loaded:
{{Command|modprobe g_ether}}

On the host machine, make sure the usbnet module is loaded:
{{Command|modprobe usbnet}}

Now you can connect both computers with the USB cable.

After a second or so, on the host machine run the following command, changing the prefix "192.168.0." to whatever was set up in your dnsmasq configuration previously:
{{Command|ifconfig usb0 192.168.0.1 netmask 255.255.255.0 broadcast 192.168.0.255}}

On the client, start dhcpcd:
{{Command|dhcpcd usb0}}

=== Troubleshooting ===
If you did not get the '192.168.xxx.xxx' lease, then it did not work (which happens on a routine basis with some of our developers). Follow these instructions to make sure everything is working correctly.

First, you'll want to stop the current dhcpcd daemon so you can start it again later when you need to:
{{Command|dhcpcd -k}}

Verify that DHCPREQUEST was being sent by searching for any recent DHCPREQUEST messages in /var/log/messages.

Reconnect, use the ifconfig command for the host again, then '''immediately after the ifconfig command completes''' run dhcpcd on the client again.

== Bridging ==

=== Prerequisites ===

*Linux host
**Linux kernel
***Bridging support
***usbnet (usb networking)

*Networking medium
**Networking hub (ethernet, wireless)
**Networking cable (usb, serial)

*Linux client
**Linux Kernel
***g_ether

=== Networking Overview ===
For the purposes of this HOWTO we are assuming that the host is connect via Ethernet on a working external network and that we are connecting to it via USB.  All portions of this HOWTO under [[#Host Setup]] are commands designated to be executed on the bridging host.  All items on [[#Client Setup]] are commands to be executed on your client device.

*Host
**eth0: external networked device
**usb0: route to connecting device
**br0: host bridge

*Client
**usb0: route to bridging host

=== Host Setup ===
You'll need the proper Linux support:
{{Kernel|2.6.28-gentoo-r5|<pre>
 Networking Support --->
   Networking Options --->
      <*> 802.1d Ethernet Bridging
 Device Drivers --->
   Network device support --->
     USB Network Adapters --->
       -*- Multi-purpose USB Networking Framework
       <M> Simple USB Networking Links (CDC Ethernet subset)
</pre>}}

You'll need to set up the bridge:
{{Command|emerge -n net-misc/bridge-utils}}
{{Command|cd /etc/init.d}}
{{Command|ln -s net.lo net.usb0}}
{{Command|ln -s net.lo net.br0}}

'''baselayout-1'''
{{File|/etc/conf.d/net|<pre>
#Bridge
bridge_br0="eth0"
bridge_add_usb0="br0"
config_eth0=( "null" )
config_usb0=( "null" )
config_br0=( "dhcp" )
depend_br0() {
  need net.eth0
}
</pre>}}

'''baselayout-2'''

{{File|/etc/conf.d/net|<pre>
#Bridge
bridge_br0="eth0"
bridge_add_usb0="br0"
config_eth0="null"
config_usb0="null"
config_br0="dhcp"
depend_br0() {
  need net.eth0
}
</pre>}}

=== Client Setup ===
{{Kernel|2.6.28-omap1|<pre>
 Device Drivers --->
   Network device support --->
     USB Support --->
       <M> USB Gadget Support --->
         <M> USB Gadget Drivers
         <M> Ethernet Gadget (with CDC Ethernet support)
         [*] RNDIS support
</pre>}}

=== Starting the Network  ===

==== Host ====
Start the bridge:
{{Command|rc-update del net.eth0 default}}
{{Command|rc-update add net.br0 default}}
{{Command|/etc/init.d/net.eth0 pause}}
{{Command|/etc/init.d/net.br0 start}}

Load appropriate modules if you didn't compile into kernel.
{{Command|modprobe -k cdc_ether}}

If you are not using DHCP, then establish static IP
{{Command|ifconfig usb0 192.168.0.201}}

==== Client ====
''Static IP''
----
Load appropriate modules if you didn't compile into kernel.
{{Command|modprobe -k g_ether}}
{{Command|ifconfig usb0 192.168.0.202}}

''DHCP''
----
Load module and set MAC addr
{{Command|<nowiki>modprobe -k g_ether dev_addr=00:AA:AA:AA:AA:01</nowiki>}}
{{Command|dhcpcd -h neuvoo usb0}}

=== Troubleshooting ===

==== How do I know that my bridged devices are on the network? ====
{{Command|brctl show}}
{{Code|brctl output|<pre>
bridge name	bridge id		STP enabled	interfaces
br0		8000.0022152c7e46	no		eth0
							usb0
</pre>}}

==== How do I know that my device is detected? ====
{{Command|lsusb}}
{{Code|lsusb output|<pre>
Bus 001 Device 005: ID 0525:a4a2 Netchip Technology, Inc. Linux-USB Ethernet/RNDIS Gadget
</pre>}}