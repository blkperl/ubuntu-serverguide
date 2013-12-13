Networking
==========

Networks consist of two or more devices, such as computer systems,
printers, and related equipment which are connected by either physical
cabling or wireless links for the purpose of sharing and distributing
information among the connected devices.

This section provides general and specific information pertaining to
networking, including an overview of network concepts and detailed
discussion of popular network protocols.

Network Configuration
=====================

Ubuntu ships with a number of graphical utilities to configure your
network devices. This document is geared toward server administrators
and will focus on managing your network on the command line.

Ethernet Interfaces
-------------------

Ethernet interfaces are identified by the system using the naming
convention of *ethX*, where *X* represents a numeric value. The first
Ethernet interface is typically identified as *eth0*, the second as
*eth1*, and all others should move up in numerical order.

Identify Ethernet Interfaces
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To quickly identify all available Ethernet interfaces, you can use the
ifconfig command as shown below.

::


Another application that can help identify all network interfaces
available to your system is the lshw command. In the example below, lshw
shows a single Ethernet interface with the logical name of *eth0* along
with bus information, driver details and all supported capabilities.

::


Ethernet Interface Logical Names
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Interface logical names are configured in the file
``/etc/udev/rules.d/70-persistent-net.rules.`` If you would like control
which interface receives a particular logical name, find the line
matching the interfaces physical MAC address and modify the value of
*NAME=ethX* to the desired logical name. Reboot the system to commit
your changes.

Ethernet Interface Settings
~~~~~~~~~~~~~~~~~~~~~~~~~~~

ethtool is a program that displays and changes Ethernet card settings
such as auto-negotiation, port speed, duplex mode, and Wake-on-LAN. It
is not installed by default, but is available for installation in the
repositories.

::

The following is an example of how to view supported features and
configured settings of an Ethernet interface.

::


Changes made with the ethtool command are temporary and will be lost
after a reboot. If you would like to retain settings, simply add the
desired ethtool command to a *pre-up* statement in the interface
configuration file ``/etc/network/interfaces``.

The following is an example of how the interface identified as *eth0*
could be permanently configured with a port speed of 1000Mb/s running in
full duplex mode.

::

    auto eth0
    iface eth0 inet static
    pre-up /sbin/ethtool -s eth0 speed 1000 duplex full

    **Note**

    Although the example above shows the interface configured to use the
    *static* method, it actually works with other methods as well, such
    as DHCP. The example is meant to demonstrate only proper placement
    of the *pre-up* statement in relation to the rest of the interface
    configuration.

IP Addressing
-------------

The following section describes the process of configuring your systems
IP address and default gateway needed for communicating on a local area
network and the Internet.

Temporary IP Address Assignment
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

For temporary network configurations, you can use standard commands such
as ip, ifconfig and route, which are also found on most other GNU/Linux
operating systems. These commands allow you to configure settings which
take effect immediately, however they are not persistent and will be
lost after a reboot.

To temporarily configure an IP address, you can use the ifconfig command
in the following manner. Just modify the IP address and subnet mask to
match your network requirements.

::

To verify the IP address configuration of eth0, you can use the ifconfig
command in the following manner.

::


     

To configure a default gateway, you can use the route command in the
following manner. Modify the default gateway address to match your
network requirements.

::

To verify your default gateway configuration, you can use the route
command in the following manner.

::


If you require DNS for your temporary network configuration, you can add
DNS server IP addresses in the file ``/etc/resolv.conf``. In general,
editing ``/etc/resolv.conf`` directly is not recommanded, but this is a
temporary and non-peristent configuration. The example below shows how
to enter two DNS servers to ``/etc/resolv.conf``, which should be
changed to servers appropriate for your network. A more lengthy
description of the proper persistent way to do DNS client configuration
is in a following section.

::

    nameserver 8.8.8.8
    nameserver 8.8.4.4

If you no longer need this configuration and wish to purge all IP
configuration from an interface, you can use the ip command with the
flush option as shown below.

::

    **Note**

    Flushing the IP configuration using the ip command does not clear
    the contents of ``/etc/resolv.conf``. You must remove or modify
    those entries manually, or re-boot which should also cause
    ``/etc/resolv.conf``, which is actually now a symlink to
    ``/run/resolvconf/resolv.conf``, to be re-written.

Dynamic IP Address Assignment (DHCP Client)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To configure your server to use DHCP for dynamic address assignment, add
the *dhcp* method to the inet address family statement for the
appropriate interface in the file ``/etc/network/interfaces``. The
example below assumes you are configuring your first Ethernet interface
identified as *eth0*.

::

    auto eth0
    iface eth0 inet dhcp

By adding an interface configuration as shown above, you can manually
enable the interface through the ifup command which initiates the DHCP
process via dhclient.

::

To manually disable the interface, you can use the ifdown command, which
in turn will initiate the DHCP release process and shut down the
interface.

::

Static IP Address Assignment
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To configure your system to use a static IP address assignment, add the
*static* method to the inet address family statement for the appropriate
interface in the file ``/etc/network/interfaces``. The example below
assumes you are configuring your first Ethernet interface identified as
*eth0*. Change the *address*, *netmask*, and *gateway* values to meet
the requirements of your network.

::

    auto eth0
    iface eth0 inet static
    address 10.0.0.100
    netmask 255.255.255.0
    gateway 10.0.0.1

By adding an interface configuration as shown above, you can manually
enable the interface through the ifup command.

::

To manually disable the interface, you can use the ifdown command.

::

Loopback Interface
~~~~~~~~~~~~~~~~~~

The loopback interface is identified by the system as *lo* and has a
default IP address of 127.0.0.1. It can be viewed using the ifconfig
command.

::


By default, there should be two lines in ``/etc/network/interfaces``
responsible for automatically configuring your loopback interface. It is
recommended that you keep the default settings unless you have a
specific purpose for changing them. An example of the two default lines
are shown below.

::

    auto lo
    iface lo inet loopback

Name Resolution
---------------

Name resolution as it relates to IP networking is the process of mapping
IP addresses to hostnames, making it easier to identify resources on a
network. The following section will explain how to properly configure
your system for name resolution using DNS and static hostname records.

DNS Client Configuration
~~~~~~~~~~~~~~~~~~~~~~~~

Traditionally, the file ``/etc/resolv.conf`` was a static configuration
file that rarely needed to be changed or automatically changed via DCHP
client hooks. Nowadays, a computer can switch from one network to
another quite often and the *resolvconf* framework is now being used to
track these changes and update the resolver's configuration
automatically. It acts as an intermediary between programs that supply
nameserver information and applications that need nameserver
information. Resolvconf gets populated with information by a set of hook
scripts related to network interface configuration. The most notable
difference for the user is that any change manually done to
``/etc/resolv.conf`` will be lost as it gets overwritten each time
something triggers resolvconf. Instead, resolvconf uses DHCP client
hooks, and ``/etc/network/interfaces`` to generate a list of nameservers
and domains to put in ``/etc/resolv.conf``, which is now a symlink:

::

    /etc/resolv.conf -> ../run/resolvconf/resolv.conf

To configure the resolver, add the IP addresses of the nameservers that
are appropriate for your network in the file
``/etc/network/interfaces``. You can also add an optional DNS suffix
search-lists to match your network domain names. For each other valid
resolv.conf configuration option, you can include, in the stanza, one
line beginning with that option name with a **dns-** prefix. The
resulting file might look like the following:

::

    iface eth0 inet static
        address 192.168.3.3
        netmask 255.255.255.0
        gateway 192.168.3.1
        dns-search example.com
        dns-nameservers 192.168.3.45 192.168.8.10

The *search* option can also be used with multiple domain names so that
DNS queries will be appended in the order in which they are entered. For
example, your network may have multiple sub-domains to search; a parent
domain of *example.com*, and two sub-domains, *sales.example.com* and
*dev.example.com*.

If you have multiple domains you wish to search, your configuration
might look like the following:

::

    iface eth0 inet static
        address 192.168.3.3
        netmask 255.255.255.0
        gateway 192.168.3.1
        dns-search example.com sales.example.com dev.example.com
        dns-nameservers 192.168.3.45 192.168.8.10

If you try to ping a host with the name of *server1*, your system will
automatically query DNS for its Fully Qualified Domain Name (FQDN) in
the following order:

1. server1\ **.example.com**

2. server1\ **.sales.example.com**

3. server1\ **.dev.example.com**

If no matches are found, the DNS server will provide a result of
*notfound* and the DNS query will fail.

Static Hostnames
~~~~~~~~~~~~~~~~

Static hostnames are locally defined hostname-to-IP mappings located in
the file ``/etc/hosts``. Entries in the ``hosts`` file will have
precedence over DNS by default. This means that if your system tries to
resolve a hostname and it matches an entry in /etc/hosts, it will not
attempt to look up the record in DNS. In some configurations, especially
when Internet access is not required, servers that communicate with a
limited number of resources can be conveniently set to use static
hostnames instead of DNS.

The following is an example of a ``hosts`` file where a number of local
servers have been identified by simple hostnames, aliases and their
equivalent Fully Qualified Domain Names (FQDN's).

::

    127.0.0.1   localhost
    127.0.1.1   ubuntu-server
    10.0.0.11   server1 server1.example.com vpn
    10.0.0.12   server2 server2.example.com mail
    10.0.0.13   server3 server3.example.com www
    10.0.0.14   server4 server4.example.com file

    **Note**

    In the above example, notice that each of the servers have been
    given aliases in addition to their proper names and FQDN's.
    *Server1* has been mapped to the name *vpn*, *server2* is referred
    to as *mail*, *server3* as *www*, and *server4* as *file*.

Name Service Switch Configuration
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The order in which your system selects a method of resolving hostnames
to IP addresses is controlled by the Name Service Switch (NSS)
configuration file ``/etc/nsswitch.conf``. As mentioned in the previous
section, typically static hostnames defined in the systems
``/etc/hosts`` file have precedence over names resolved from DNS. The
following is an example of the line responsible for this order of
hostname lookups in the file ``/etc/nsswitch.conf``.

::

    hosts:          files mdns4_minimal [NOTFOUND=return] dns mdns4

-  **files** first tries to resolve static hostnames located in
   ``/etc/hosts``.

-  **mdns4\_minimal** attempts to resolve the name using Multicast DNS.

-  **[NOTFOUND=return]** means that any response of *notfound* by the
   preceding *mdns4\_minimal* process should be treated as authoritative
   and that the system should not try to continue hunting for an answer.

-  **dns** represents a legacy unicast DNS query.

-  **mdns4** represents a Multicast DNS query.

To modify the order of the above mentioned name resolution methods, you
can simply change the *hosts:* string to the value of your choosing. For
example, if you prefer to use legacy Unicast DNS versus Multicast DNS,
you can change the string in ``/etc/nsswitch.conf`` as shown below.

::

    hosts:          files dns [NOTFOUND=return] mdns4_minimal mdns4

Bridging
--------

Bridging multiple interfaces is a more advanced configuration, but is
very useful in multiple scenarios. One scenario is setting up a bridge
with multiple network interfaces, then using a firewall to filter
traffic between two network segments. Another scenario is using bridge
on a system with one interface to allow virtual machines direct access
to the outside network. The following example covers the latter
scenario.

Before configuring a bridge you will need to install the bridge-utils
package. To install the package, in a terminal enter:

::

Next, configure the bridge by editing ``/etc/network/interfaces``:

::

    auto lo
    iface lo inet loopback

    auto br0
    iface br0 inet static
            address 192.168.0.10
            network 192.168.0.0
            netmask 255.255.255.0
            broadcast 192.168.0.255
            gateway 192.168.0.1
            bridge_ports eth0
            bridge_fd 9
            bridge_hello 2
            bridge_maxage 12
            bridge_stp off

    **Note**

    Enter the appropriate values for your physical interface and
    network.

Now restart networking to enable the bridge interface:

::

The new bridge interface should now be up and running. The brctl
provides useful information about the state of the bridge, controls
which interfaces are part of the bridge, etc. See ``man brctl`` for more
information.

Resources
---------

-  The `Ubuntu Wiki Network
   page <https://help.ubuntu.com/community/Network>`__ has links to
   articles covering more advanced network configuration.

-  The `resolvconf man
   page <http://manpages.ubuntu.com/manpages/man8/resolvconf.8.html>`__
   has more information on resolvconf.

-  The `interfaces man
   page <http://manpages.ubuntu.com/manpages/man5/interfaces.5.html>`__
   has details on more options for ``/etc/network/interfaces``.

-  The `dhclient man
   page <http://manpages.ubuntu.com/manpages/man8/dhclient.8.html>`__
   has details on more options for configuring DHCP client settings.

-  For more information on DNS client configuration see the `resolver
   man
   page <http://manpages.ubuntu.com/manpages/man5/resolver.5.html>`__.
   Also, Chapter 6 of O'Reilly's `Linux Network Administrator's
   Guide <http://oreilly.com/catalog/linag2/book/ch06.html>`__ is a good
   source of resolver and name service configuration information.

-  For more information on *bridging* see the `brctl man
   page <http://manpages.ubuntu.com/manpages/man8/brctl.8.html>`__ and
   the Linux Foundation's
   `Networking-Bridge <http://www.linuxfoundation.org/collaborate/workgroups/networking/bridge>`__
   page.

TCP/IP
======

The Transmission Control Protocol and Internet Protocol (TCP/IP) is a
standard set of protocols developed in the late 1970s by the Defense
Advanced Research Projects Agency (DARPA) as a means of communication
between different types of computers and computer networks. TCP/IP is
the driving force of the Internet, and thus it is the most popular set
of network protocols on Earth.

TCP/IP Introduction
-------------------

The two protocol components of TCP/IP deal with different aspects of
computer networking. *Internet Protocol*, the "IP" of TCP/IP is a
connectionless protocol which deals only with network packet routing
using the *IP Datagram* as the basic unit of networking information. The
IP Datagram consists of a header followed by a message. The
*Transmission Control Protocol* is the "TCP" of TCP/IP and enables
network hosts to establish connections which may be used to exchange
data streams. TCP also guarantees that the data between connections is
delivered and that it arrives at one network host in the same order as
sent from another network host.

TCP/IP Configuration
--------------------

The TCP/IP protocol configuration consists of several elements which
must be set by editing the appropriate configuration files, or deploying
solutions such as the Dynamic Host Configuration Protocol (DHCP) server
which in turn, can be configured to provide the proper TCP/IP
configuration settings to network clients automatically. These
configuration values must be set correctly in order to facilitate the
proper network operation of your Ubuntu system.

The common configuration elements of TCP/IP and their purposes are as
follows:

-  **IP address** The IP address is a unique identifying string
   expressed as four decimal numbers ranging from zero (0) to
   two-hundred and fifty-five (255), separated by periods, with each of
   the four numbers representing eight (8) bits of the address for a
   total length of thirty-two (32) bits for the whole address. This
   format is called *dotted quad notation*.

-  **Netmask** The Subnet Mask (or simply, *netmask*) is a local bit
   mask, or set of flags which separate the portions of an IP address
   significant to the network from the bits significant to the
   *subnetwork*. For example, in a Class C network, the standard netmask
   is 255.255.255.0 which masks the first three bytes of the IP address
   and allows the last byte of the IP address to remain available for
   specifying hosts on the subnetwork.

-  **Network Address** The Network Address represents the bytes
   comprising the network portion of an IP address. For example, the
   host 12.128.1.2 in a Class A network would use 12.0.0.0 as the
   network address, where twelve (12) represents the first byte of the
   IP address, (the network part) and zeroes (0) in all of the remaining
   three bytes to represent the potential host values. A network host
   using the private IP address 192.168.1.100 would in turn use a
   Network Address of 192.168.1.0, which specifies the first three bytes
   of the Class C 192.168.1 network and a zero (0) for all the possible
   hosts on the network.

-  **Broadcast Address** The Broadcast Address is an IP address which
   allows network data to be sent simultaneously to all hosts on a given
   subnetwork rather than specifying a particular host. The standard
   general broadcast address for IP networks is 255.255.255.255, but
   this broadcast address cannot be used to send a broadcast message to
   every host on the Internet because routers block it. A more
   appropriate broadcast address is set to match a specific subnetwork.
   For example, on the private Class C IP network, 192.168.1.0, the
   broadcast address is 192.168.1.255. Broadcast messages are typically
   produced by network protocols such as the Address Resolution Protocol
   (ARP) and the Routing Information Protocol (RIP).

-  **Gateway Address** A Gateway Address is the IP address through which
   a particular network, or host on a network, may be reached. If one
   network host wishes to communicate with another network host, and
   that host is not located on the same network, then a *gateway* must
   be used. In many cases, the Gateway Address will be that of a router
   on the same network, which will in turn pass traffic on to other
   networks or hosts, such as Internet hosts. The value of the Gateway
   Address setting must be correct, or your system will not be able to
   reach any hosts beyond those on the same network.

-  **Nameserver Address** Nameserver Addresses represent the IP
   addresses of Domain Name Service (DNS) systems, which resolve network
   hostnames into IP addresses. There are three levels of Nameserver
   Addresses, which may be specified in order of precedence: The
   *Primary* Nameserver, the *Secondary* Nameserver, and the *Tertiary*
   Nameserver. In order for your system to be able to resolve network
   hostnames into their corresponding IP addresses, you must specify
   valid Nameserver Addresses which you are authorized to use in your
   system's TCP/IP configuration. In many cases these addresses can and
   will be provided by your network service provider, but many free and
   publicly accessible nameservers are available for use, such as the
   Level3 (Verizon) servers with IP addresses from 4.2.2.1 to 4.2.2.6.

       **Tip**

       The IP address, Netmask, Network Address, Broadcast Address,
       Gateway Address, and Nameserver Addresses are typically specified
       via the appropriate directives in the file
       ``/etc/network/interfaces``. For more information, view the
       system manual page for ``interfaces``, with the following command
       typed at a terminal prompt:

   Access the system manual page for ``interfaces`` with the following
   command:

   ::

IP Routing
----------

IP routing is a means of specifying and discovering paths in a TCP/IP
network along which network data may be sent. Routing uses a set of
*routing tables* to direct the forwarding of network data packets from
their source to the destination, often via many intermediary network
nodes known as *routers*. There are two primary forms of IP routing:
*Static Routing* and *Dynamic Routing.*

Static routing involves manually adding IP routes to the system's
routing table, and this is usually done by manipulating the routing
table with the route command. Static routing enjoys many advantages over
dynamic routing, such as simplicity of implementation on smaller
networks, predictability (the routing table is always computed in
advance, and thus the route is precisely the same each time it is used),
and low overhead on other routers and network links due to the lack of a
dynamic routing protocol. However, static routing does present some
disadvantages as well. For example, static routing is limited to small
networks and does not scale well. Static routing also fails completely
to adapt to network outages and failures along the route due to the
fixed nature of the route.

Dynamic routing depends on large networks with multiple possible IP
routes from a source to a destination and makes use of special routing
protocols, such as the Router Information Protocol (RIP), which handle
the automatic adjustments in routing tables that make dynamic routing
possible. Dynamic routing has several advantages over static routing,
such as superior scalability and the ability to adapt to failures and
outages along network routes. Additionally, there is less manual
configuration of the routing tables, since routers learn from one
another about their existence and available routes. This trait also
eliminates the possibility of introducing mistakes in the routing tables
via human error. Dynamic routing is not perfect, however, and presents
disadvantages such as heightened complexity and additional network
overhead from router communications, which does not immediately benefit
the end users, but still consumes network bandwidth.

TCP and UDP
-----------

TCP is a connection-based protocol, offering error correction and
guaranteed delivery of data via what is known as *flow control*. Flow
control determines when the flow of a data stream needs to be stopped,
and previously sent data packets should to be re-sent due to problems
such as *collisions*, for example, thus ensuring complete and accurate
delivery of the data. TCP is typically used in the exchange of important
information such as database transactions.

The User Datagram Protocol (UDP), on the other hand, is a
*connectionless* protocol which seldom deals with the transmission of
important data because it lacks flow control or any other method to
ensure reliable delivery of the data. UDP is commonly used in such
applications as audio and video streaming, where it is considerably
faster than TCP due to the lack of error correction and flow control,
and where the loss of a few packets is not generally catastrophic.

ICMP
----

The Internet Control Messaging Protocol (ICMP) is an extension to the
Internet Protocol (IP) as defined in the Request For Comments (RFC) #792
and supports network packets containing control, error, and
informational messages. ICMP is used by such network applications as the
ping utility, which can determine the availability of a network host or
device. Examples of some error messages returned by ICMP which are
useful to both network hosts and devices such as routers, include
*Destination Unreachable* and *Time Exceeded*.

Daemons
-------

Daemons are special system applications which typically execute
continuously in the background and await requests for the functions they
provide from other applications. Many daemons are network-centric; that
is, a large number of daemons executing in the background on an Ubuntu
system may provide network-related functionality. Some examples of such
network daemons include the *Hyper Text Transport Protocol Daemon*
(httpd), which provides web server functionality; the *Secure SHell
Daemon* (sshd), which provides secure remote login shell and file
transfer capabilities; and the *Internet Message Access Protocol Daemon*
(imapd), which provides E-Mail services.

Resources
---------

-  There are man pages for
   `TCP <http://manpages.ubuntu.com/manpages/&distro-short-codename;/en/man7/tcp.7.html>`__
   and
   `IP <http://manpages.ubuntu.com/manpages/&distro-short-codename;/man7/ip.7.html>`__
   that contain more useful information.

-  Also, see the `TCP/IP Tutorial and Technical
   Overview <http://www.redbooks.ibm.com/abstracts/gg243376.html>`__ IBM
   Redbook.

-  Another resource is O'Reilly's `TCP/IP Network
   Administration <http://oreilly.com/catalog/9780596002978/>`__.

Dynamic Host Configuration Protocol (DHCP)
==========================================

The Dynamic Host Configuration Protocol (DHCP) is a network service that
enables host computers to be automatically assigned settings from a
server as opposed to manually configuring each network host. Computers
configured to be DHCP clients have no control over the settings they
receive from the DHCP server, and the configuration is transparent to
the computer's user.

The most common settings provided by a DHCP server to DHCP clients
include:

-  IP address and netmask

-  IP address of the default-gateway to use

-  IP adresses of the DNS servers to use

However, a DHCP server can also supply configuration properties such as:

-  Host Name

-  Domain Name

-  Time Server

-  Print Server

The advantage of using DHCP is that changes to the network, for example
a change in the address of the DNS server, need only be changed at the
DHCP server, and all network hosts will be reconfigured the next time
their DHCP clients poll the DHCP server. As an added advantage, it is
also easier to integrate new computers into the network, as there is no
need to check for the availability of an IP address. Conflicts in IP
address allocation are also reduced.

A DHCP server can provide configuration settings using the following
methods:

Manual allocation (MAC address)
    This method entails using DHCP to identify the unique hardware
    address of each network card connected to the network and then
    continually supplying a constant configuration each time the DHCP
    client makes a request to the DHCP server using that network device.
    This ensures that a particular address is assigned automatically to
    that network card, based on it's MAC address.

Dynamic allocation (address pool)
    In this method, the DHCP server will assign an IP address from a
    pool of addresses (sometimes also called a range or scope) for a
    period of time or lease, that is configured on the server or until
    the client informs the server that it doesn't need the address
    anymore. This way, the clients will be receiving their configuration
    properties dynamically and on a "first come, first served" basis.
    When a DHCP client is no longer on the network for a specified
    period, the configuration is expired and released back to the
    address pool for use by other DHCP Clients. This way, an address can
    be leased or used for a period of time. After this period, the
    client has to renegociate the lease with the server to maintain use
    of the address.

Automatic allocation
    Using this method, the DHCP automatically assigns an IP address
    permanently to a device, selecting it from a pool of available
    addresses. Usually DHCP is used to assign a temporary address to a
    client, but a DHCP server can allow an infinite lease time.

The last two methods can be considered "automatic" because in each case
the DHCP server assigns an address with no extra intervention needed.
The only difference between them is in how long the IP address is
leased, in other words whether a client's address varies over time.
Ubuntu is shipped with both DHCP server and client. The server is dhcpd
(dynamic host configuration protocol daemon). The client provided with
Ubuntu is dhclient and should be installed on all computers required to
be automatically configured. Both programs are easy to install and
configure and will be automatically started at system boot.

Installation
------------

At a terminal prompt, enter the following command to install dhcpd:

::

You will probably need to change the default configuration by editing
/etc/dhcp/dhcpd.conf to suit your needs and particular configuration.

You also may need to edit /etc/default/isc-dhcp-server to specify the
interfaces dhcpd should listen to.

NOTE: dhcpd's messages are being sent to syslog. Look there for
diagnostics messages.

Configuration
-------------

The error message the installation ends with might be a little
confusing, but the following steps will help you configure the service:

Most commonly, what you want to do is assign an IP address randomly.
This can be done with settings as follows:

::

    # minimal sample /etc/dhcp/dhcpd.conf
    default-lease-time 600;
    max-lease-time 7200;

    subnet 192.168.1.0 netmask 255.255.255.0 {
     range 192.168.1.150 192.168.1.200;
     option routers 192.168.1.254;
     option domain-name-servers 192.168.1.1, 192.168.1.2;
     option domain-name "mydomain.example";
    } 

This will result in the DHCP server giving clients an IP address from
the range 192.168.1.150-192.168.1.200. It will lease an IP address for
600 seconds if the client doesn't ask for a specific time frame.
Otherwise the maximum (allowed) lease will be 7200 seconds. The server
will also "advise" the client to use 192.168.1.254 as the
default-gateway and 192.168.1.1 and 192.168.1.2 as its DNS servers.

After changing the config file you have to restart the dhcpd:

::

References
----------

-  The `dhcp3-server Ubuntu
   Wiki <https://help.ubuntu.com/community/dhcp3-server>`__ page has
   more information.

-  For more ``/etc/dhcp/dhcpd.conf`` options see the `dhcpd.conf man
   page <http://manpages.ubuntu.com/manpages/&distro-short-codename;/en/man5/dhcpd.conf.5.html>`__.

-  `ISC dhcp-server <http://www.isc.org/software/dhcp>`__

Time Synchronisation with NTP
=============================

NTP is a TCP/IP protocol for synchronising time over a network.
Basically a client requests the current time from a server, and uses it
to set its own clock.

Behind this simple description, there is a lot of complexity - there are
tiers of NTP servers, with the tier one NTP servers connected to atomic
clocks, and tier two and three servers spreading the load of actually
handling requests across the Internet. Also the client software is a lot
more complex than you might think - it has to factor out communication
delays, and adjust the time in a way that does not upset all the other
processes that run on the server. But luckily all that complexity is
hidden from you!

Ubuntu uses ntpdate and ntpd.

ntpdate
-------

Ubuntu comes with ntpdate as standard, and will run it once at boot time
to set up your time according to Ubuntu's NTP server.

::

    ntpdate -s ntp.ubuntu.com

ntpd
----

The ntp daemon ntpd calculates the drift of your system clock and
continuously adjusts it, so there are no large corrections that could
lead to inconsistent logs for instance. The cost is a little processing
power and memory, but for a modern server this is negligible.

Installation
------------

To install ntpd, from a terminal prompt enter:

::

Configuration
-------------

Edit ``/etc/ntp.conf`` to add/remove server lines. By default these
servers are configured:

::

    # Use servers from the NTP Pool Project. Approved by Ubuntu Technical Board
    # on 2011-02-08 (LP: #104525). See http://www.pool.ntp.org/join.html for
    # more information.
    server 0.ubuntu.pool.ntp.org
    server 1.ubuntu.pool.ntp.org
    server 2.ubuntu.pool.ntp.org
    server 3.ubuntu.pool.ntp.org

After changing the config file you have to reload the ntpd:

::

View status
-----------

Use ntpq to see more info:

::


References
----------

-  See the `Ubuntu
   Time <https://help.ubuntu.com/community/UbuntuTime>`__ wiki page for
   more information.

-  `ntp.org, home of the Network Time Protocol
   project <http://www.ntp.org/>`__


