VPN
===

OpenVPN is a Virtual Private Networking (VPN) solution provided in the
Ubuntu Repositories. It is flexible, reliable and secure. It belongs to
the family of SSL/TLS VPN stacks (different from IPSec VPNs). This
chapter will cover installing and configuring OpenVPN to create a VPN.

OpenVPN
=======

If you want more than just pre-shared keys OpenVPN makes it easy to
setup and use a Public Key Infrastructure (PKI) to use SSL/TLS
certificates for authentication and key exchange between the VPN server
and clients. OpenVPN can be used in a routed or bridged VPN mode and can
be configured to use either UDP or TCP. The port number can be
configured as well, but port 1194 is the official one. And it is only
using that single port for all communication. VPN client implementations
are available for almost anything including all Linux distributions, OS
X, Windows and OpenWRT based WLAN routers.

Server Installation
-------------------

To install openvpn in a terminal enter:

::

Public Key Infrastructure Setup
-------------------------------

The first step in building an OpenVPN configuration is to establish a
PKI (public key infrastructure). The PKI consists of:

-  a separate certificate (also known as a public key) and private key
   for the server and each client, and

-  a master Certificate Authority (CA) certificate and key which is used
   to sign each of the server and client certificates.

OpenVPN supports bidirectional authentication based on certificates,
meaning that the client must authenticate the server certificate and the
server must authenticate the client certificate before mutual trust is
established.

Both server and client will authenticate the other by first verifying
that the presented certificate was signed by the master certificate
authority (CA), and then by testing information in the now-authenticated
certificate header, such as the certificate common name or certificate
type (client or server).

Certificate Authority Setup
~~~~~~~~~~~~~~~~~~~~~~~~~~~

To setup your own Certificate Authority (CA) and generating certificates
and keys for an OpenVPN server and multiple clients first copy the
``easy-rsa`` directory to ``/etc/openvpn``. This will ensure that any
changes to the scripts will not be lost when the package is updated.
From a terminal change to user root and:

::


Next, edit ``/etc/openvpn/easy-rsa/vars`` adjusting the following to
your environment:

::

    export KEY_COUNTRY="US"
    export KEY_PROVINCE="NC"
    export KEY_CITY="Winston-Salem"
    export KEY_ORG="Example Company"
    export KEY_EMAIL="steve@example.com"

Enter the following to generate the master Certificate Authority (CA)
certificate and key:

::




Server Certificates
~~~~~~~~~~~~~~~~~~~

Next, we will generate a certificate and private key for the server:

::

As in the previous step, most parameters can be defaulted. Two other
queries require positive responses, "Sign the certificate? [y/n]" and "1
out of 1 certificate requests certified, commit? [y/n]".

Diffie Hellman parameters must be generated for the OpenVPN server:

::

All certificates and keys have been generated in the subdirectory keys/.
Common practice is to copy them to /etc/openvpn/:

::


Client Certificates
~~~~~~~~~~~~~~~~~~~

The VPN client will also need a certificate to authenticate itself to
the server. Usually you create a different certificate for each client.
To create the certificate, enter the following in a terminal while being
user root:

::



Copy the following files to the client using a secure method:

-  /etc/openvpn/ca.crt

-  /etc/openvpn/easy-rsa/keys/client1.crt

-  /etc/openvpn/easy-rsa/keys/client1.key

As the client certificates and keys are only required on the client
machine, you should remove them from the server.

Simple Server Configuration
---------------------------

Along with your OpenVPN installation you got these sample config files
(and many more if if you check):

::

    root@server:/# ls -l /usr/share/doc/openvpn/examples/sample-config-files/
    total 68
    -rw-r--r-- 1 root root 3427 2011-07-04 15:09 client.conf
    -rw-r--r-- 1 root root 4141 2011-07-04 15:09 server.conf.gz

Start with copying and unpacking server.conf.gz to
/etc/openvpn/server.conf.

::


Edit ``/etc/openvpn/server.conf`` to make sure the following lines are
pointing to the certificates and keys you created in the section above.

::

    ca ca.crt
    cert myservername.crt
    key myservername.key 
    dh dh1024.pem

That is the minimum you have to configure to get a working OpenVPN
server. You can use all the default settings in the sample server.conf
file. Now start the server. You will find logging and error messages in
your syslog.

::

    root@server:/etc/openvpn# service openvpn start
     * Starting virtual private network daemon(s)...
       *   Autostarting VPN 'server'                     [ OK ]

Now check if OpenVPN created a tun0 interface:

::

    root@server:/etc/openvpn# ifconfig tun0
    tun0      Link encap:UNSPEC  HWaddr 00-00-00-00-00-00-00-00-00-00-00-00-00-00-00-00  
              inet addr:10.8.0.1  P-t-P:10.8.0.2  Mask:255.255.255.255
              UP POINTOPOINT RUNNING NOARP MULTICAST  MTU:1500  Metric:1
    [...]

Simple Client Configuration
---------------------------

There are various different OpenVPN client implementations with and
without GUIs. You can read more about clients in a later section. For
now we use the OpenVPN client for Ubuntu which is the same executable as
the server. So you have to install the openvpn package again on the
client machine:

::

This time copy the client.conf sample config file to /etc/openvpn/.

::

Copy the client keys and the certificate of the CA you created in the
section above to e.g. /etc/openvpn/ and edit
``/etc/openvpn/client.conf`` to make sure the following lines are
pointing to those files. If you have the files in /etc/openvpn/ you can
omit the path.

::

    ca ca.crt
    cert client1.crt
    key client1.key

And you have to at least specify the OpenVPN server name or address.
Make sure the keyword client is in the config. That's what enables
client mode.

::

    client
    remote vpnserver.example.com 1194

Now start the OpenVPN client:

::

    root@client:/etc/openvpn# service openvpn start
     * Starting virtual private network daemon(s)...   
       *   Autostarting VPN 'client'                          [ OK ] 

Check if it created a tun0 interface:

::

    root@client:/etc/openvpn# ifconfig tun0
    tun0      Link encap:UNSPEC  HWaddr 00-00-00-00-00-00-00-00-00-00-00-00-00-00-00-00  
              inet addr:10.8.0.6  P-t-P:10.8.0.5  Mask:255.255.255.255
              UP POINTOPOINT RUNNING NOARP MULTICAST  MTU:1500  Metric:1

Check if you can ping the OpenVPN server:

::

    root@client:/etc/openvpn# ping 10.8.0.1
    PING 10.8.0.1 (10.8.0.1) 56(84) bytes of data.
    64 bytes from 10.8.0.1: icmp_req=1 ttl=64 time=0.920 ms

    **Note**

    The OpenVPN server always uses the first usable IP address in the
    client network and only that IP is pingable. E.g. if you configured
    a /24 for the client network mask, the .1 address will be used. The
    P-t-P address you see in the ifconfig output above is usually not
    answering ping requests.

Check out your routes:

::

    root@client:/etc/openvpn# netstat -rn
    Kernel IP routing table
    Destination     Gateway         Genmask         Flags   MSS Window  irtt Iface
    10.8.0.5        0.0.0.0         255.255.255.255 UH        0 0          0 tun0
    10.8.0.1        10.8.0.5        255.255.255.255 UGH       0 0          0 tun0
    192.168.42.0    0.0.0.0         255.255.255.0   U         0 0          0 eth0
    0.0.0.0         192.168.42.1    0.0.0.0         UG        0 0          0 eth0

First trouble shooting
----------------------

If the above didn't work for you, check this:

-  Check your syslog, e.g. grep -i vpn /var/log/syslog

-  Can the client connect to the server machine? Maybe a firewall is
   blocking access? Check syslog on server.

-  Client and server must use same protocol and port, e.g. UDP port
   1194, see port and proto config option

-  Client and server must use same config regarding compression, see
   comp-lzo config option

-  Client and server must use same config regarding bridged vs routed
   mode, see server vs server-bridge config option

Advanced configuration
----------------------

Advanced routed VPN configuration on server
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The above is a very simple working VPN. The client can access services
on the VPN server machine through an encrypted tunnel. If you want to
reach more servers or anything in other networks, push some routes to
the clients. E.g. if your company's network can be summarized to the
network 192.168.0.0/16, you could push this route to the clients. But
you will also have to change the routing for the way back - your servers
need to know a route to the VPN client-network.

Or you might push a default gateway to all the clients to send all their
internet traffic to the VPN gateway first and from there via the company
firewall into the internet. This section shows you some possible
options.

Push routes to the client to allow it to reach other private subnets
behind the server. Remember that these private subnets will also need to
know to route the OpenVPN client address pool (10.8.0.0/24) back to the
OpenVPN server.

::

    push "route 10.0.0.0 255.0.0.0"

If enabled, this directive will configure all clients to redirect their
default network gateway through the VPN, causing all IP traffic such as
web browsing and DNS lookups to go through the VPN (the OpenVPN server
machine or your central firewall may need to NAT the TUN/TAP interface
to the internet in order for this to work properly).

::

    push "redirect-gateway def1 bypass-dhcp"

Configure server mode and supply a VPN subnet for OpenVPN to draw client
addresses from. The server will take 10.8.0.1 for itself, the rest will
be made available to clients. Each client will be able to reach the
server on 10.8.0.1. Comment this line out if you are ethernet bridging.

::

    server 10.8.0.0 255.255.255.0

Maintain a record of client to virtual IP address associations in this
file. If OpenVPN goes down or is restarted, reconnecting clients can be
assigned the same virtual IP address from the pool that was previously
assigned.

::

    ifconfig-pool-persist ipp.txt

Push DNS servers to the client.

::

    push "dhcp-option DNS 10.0.0.2"
    push "dhcp-option DNS 10.1.0.2"

Allow client to client communication.

::

    client-to-client

Enable compression on the VPN link.

::

    comp-lzo

The keepalive directive causes ping-like messages to be sent back and
forth over the link so that each side knows when the other side has gone
down. Ping every 1 second, assume that remote peer is down if no ping
received during a 3 second time period.

::

    keepalive 1 3

It's a good idea to reduce the OpenVPN daemon's privileges after
initialization.

::

    user nobody
    group nogroup

OpenVPN 2.0 includes a feature that allows the OpenVPN server to
securely obtain a username and password from a connecting client, and to
use that information as a basis for authenticating the client. To use
this authentication method, first add the auth-user-pass directive to
the client configuration. It will direct the OpenVPN client to query the
user for a username/password, passing it on to the server over the
secure TLS channel.

::

    # client config!
    auth-user-pass

This will tell the OpenVPN server to validate the username/password
entered by clients using the login PAM module. Useful if you have
centralized authentication with e.g. Kerberos.

::

    plugin /usr/lib/openvpn/openvpn-auth-pam.so login

    **Note**

    Please read the OpenVPN `hardening security
    guide <http://openvpn.net/index.php/open-source/documentation/howto.html#security>`__
    for further security advice.

Advanced bridged VPN configuration on server
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

OpenVPN can be setup for either a routed or a bridged VPN mode.
Sometimes this is also referred to as OSI layer-2 versus layer-3 VPN. In
a bridged VPN all layer-2 frames - e.g. all ethernet frames - are sent
to the VPN partners and in a routed VPN only layer-3 packets are sent to
VPN partners. In bridged mode all traffic including traffic which was
traditionally LAN-local like local network broadcasts, DHCP requests,
ARP requests etc. are sent to VPN partners whereas in routed mode this
would be filtered.

Prepare interface config for bridging on server
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Make sure you have the bridge-utils package installed:

::

Before you setup OpenVPN in bridged mode you need to change your
interface configuration. Let's assume your server has an interface eth0
connected to the internet and an interface eth1 connected to the LAN you
want to bridge. Your /etc/network/interfaces would like this:

::

    auto eth0
    iface eth0 inet static
      address 1.2.3.4
      netmask 255.255.255.248
      default 1.2.3.1

    auto eth1
    iface eth1 inet static
      address 10.0.0.4
      netmask 255.255.255.0

This straight forward interface config needs to be changed into a
bridged mode like where the config of interface eth1 moves to the new
br0 interface. Plus we configure that br0 should bridge interface eth1.
We also need to make sure that interface eth1 is always in promiscuous
mode - this tells the interface to forward all ethernet frames to the IP
stack.

::

    auto eth0
    iface eth0 inet static
      address 1.2.3.4
      netmask 255.255.255.248
      default 1.2.3.1

    auto eth1
    iface eth1 inet manual
      up ip link set $IFACE up promisc on

    auto br0
    iface br0 inet static
      address 10.0.0.4
      netmask 255.255.255.0
      bridge_ports eth1

At this point you need to restart networking. Be prepared that this
might not work as expected and that you will lose remote connectivity.
Make sure you can solve problems having local access.

::

Prepare server config for bridging
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Edit ``/etc/openvpn/server.conf`` changing the following options to:

::

    ;dev tun
    dev tap
    up "/etc/openvpn/up.sh br0 eth1"
    ;server 10.8.0.0 255.255.255.0
    server-bridge 10.0.0.4 255.255.255.0 10.0.0.128 10.0.0.254

Next, create a helper script to add the *tap* interface to the bridge
and to ensure that eth1 is promiscuous mode. Create
``/etc/openvpn/up.sh``:

::

    #!/bin/sh

    BR=$1
    ETHDEV=$2
    TAPDEV=$3

    /sbin/ip link set "$TAPDEV" up
    /sbin/ip link set "$ETHDEV" promisc on
    /sbin/brctl addif $BR $TAPDEV

Then make it executable:

::

After configuring the server, restart openvpn by entering:

::

Client Configuration
^^^^^^^^^^^^^^^^^^^^

First, install openvpn on the client:

::

Then with the server configured and the client certificates copied to
the ``/etc/openvpn/`` directory, create a client configuration file by
copying the example. In a terminal on the client machine enter:

::

Now edit ``/etc/openvpn/client.conf`` changing the following options:

::

    dev tap
    ;dev tun

Finally, restart openvpn:

::

You should now be able to connect to the remote LAN through the VPN.

Client software implementations
-------------------------------

Linux Network-Manager GUI for OpenVPN
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Many Linux distributions including Ubuntu desktop variants come with
Network Manager, a nice GUI to configure your network settings. It also
can manage your VPN connections. Make sure you have package
network-manager-openvpn installed. Here you see that the installation
installs all other required packages as well:

::

    root@client:~# apt-get install network-manager-openvpn
    Reading package lists... Done
    Building dependency tree       
    Reading state information... Done
    The following extra packages will be installed:
      liblzo2-2 libpkcs11-helper1 network-manager-openvpn-gnome openvpn
    Suggested packages:
      resolvconf
    The following NEW packages will be installed:
      liblzo2-2 libpkcs11-helper1 network-manager-openvpn
      network-manager-openvpn-gnome openvpn
    0 upgraded, 5 newly installed, 0 to remove and 631 not upgraded.
    Need to get 700 kB of archives.
    After this operation, 3,031 kB of additional disk space will be used.
    Do you want to continue [Y/n]? 

To inform network-manager about the new installed packages you will have
to restart it:

::

    root@client:~# restart network-manager 
    network-manager start/running, process 3078

Open the Network Manager GUI, select the VPN tab and then the 'Add'
button. Select OpenVPN as the VPN type in the opening requester and
press 'Create'. In the next window add the OpenVPN's server name as the
'Gateway', set 'Type' to 'Certificates (TLS)', point 'User Certificate'
to your user certificate, 'CA Certificate' to your CA certificate and
'Private Key' to your private key file. Use the advanced button to
enable compression or other special settings you set on the server. Now
try to establish your VPN.

OpenVPN with GUI for Mac OS X: Tunnelblick
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Tunnelblick is an excellent free, open source implementation of a GUI
for OpenVPN for OS X. The project's homepage is at
http://code.google.com/p/tunnelblick/. Download the latest OS X
installer from there and install it. Then put your client.ovpn config
file together with the certificates and keys in
/Users/username/Library/Application Support/Tunnelblick/Configurations/
and lauch Tunnelblick from your Application folder.

::

    # sample client.ovpn for Tunnelblick
    client
    remote blue.example.com
    port 1194
    proto udp
    dev tun
    dev-type tun
    ns-cert-type server
    reneg-sec 86400
    auth-user-pass
    auth-nocache
    auth-retry interact
    comp-lzo yes
    verb 3
    ca ca.crt
    cert client.crt
    key client.key

OpenVPN with GUI for Win 7
~~~~~~~~~~~~~~~~~~~~~~~~~~

First download and install the latest `OpenVPN Windows
Installer <http://www.openvpn.net/index.php/open-source/downloads.html>`__.
OpenVPN 2.2.1 was the latest when this was written. Additionally
download an alternative Open VPN Windows GUI. The OpenVPN MI GUI from
http://openvpn-mi-gui.inside-security.de seems to be a nice one for
Windows 7. Download the latest version. 20110624 was the latest version
when this was written.

You need to start the OpenVPN service. Goto Start > Computer > Manage >
Services and Applications > Services. Find the OpenVPN service and start
it. Set it's startup type to automatic. When you start the OpenVPN MI
GUI the first time you need to run it as an administrator. You have to
right click on it and you will see that option.

You will have to write your OpenVPN config in a textfile and place it in
C:\\Program Files\\OpenVPN\\config\\client.ovpn along with the CA
certificate. You could put the user certificate in the user's home
directory like in the follwing example.

::

    # C:\Program Files\OpenVPN\config\client.ovpn
    client
    remote server.example.com
    port 1194
    proto udp
    dev tun
    dev-type tun
    ns-cert-type server
    reneg-sec 86400
    auth-user-pass
    auth-retry interact
    comp-lzo yes
    verb 3
    ca ca.crt
    cert "C:\\Users\\username\\My Documents\\openvpn\\client.crt"
    key "C:\\Users\\username\\My Documents\\openvpn\\client.key"
    management 127.0.0.1 1194
    management-hold
    management-query-passwords
    auth-retry interact

OpenVPN for OpenWRT
~~~~~~~~~~~~~~~~~~~

OpenWRT is described as a Linux distribution for embedded devices like
WLAN router. There are certain types of WLAN routers who can be flashed
to run OpenWRT. Depending on the available memory on your OpenWRT router
you can run software like OpenVPN and you could for example build a
small inexpensive branch office router with VPN connectivity to the
central office. More info on OpenVPN on OpenWRT is
`here <http://wiki.openwrt.org/doc/howto/vpn.overview>`__. And here is
the OpenWRT project's homepage: http://openwrt.org

Log into your OpenWRT router and install OpenVPN:

::


Check out /etc/config/openvpn and put your client config in there. Copy
certificates and keys to /etc/openvpn/

::

    config openvpn client1
            option enable 1                                  
            option client 1                                  
    #       option dev tap                                   
            option dev tun  
            option proto udp   
            option ca /etc/openvpn/ca.crt                
            option cert /etc/openvpn/client.crt
            option key /etc/openvpn/client.key
            option comp_lzo 1  

Restart OpenVPN:

::

You will have to see if you need to adjust your router's routing and
firewall rules.

References
----------

-  See the `OpenVPN <http://openvpn.net/>`__ website for additional
   information.

-  `OpenVPN hardening security
   guide <http://openvpn.net/index.php/open-source/documentation/howto.html#security>`__

-  Also, Pakt's `OpenVPN: Building and Integrating Virtual Private
   Networks <http://www.packtpub.com/openvpn/book>`__ is a good
   resource.


