Domain Name Service (DNS)
=========================

Domain Name Service (DNS) is an Internet service that maps IP addresses
and fully qualified domain names (FQDN) to one another. In this way, DNS
alleviates the need to remember IP addresses. Computers that run DNS are
called *name servers*. Ubuntu ships with BIND (Berkley Internet Naming
Daemon), the most common program used for maintaining a name server on
Linux.

Installation
============

At a terminal prompt, enter the following command to install dns:

::

A very useful package for testing and troubleshooting DNS issues is the
dnsutils package. Very often these tools will be installed already, but
to check and/or install dnsutils enter the following:

::

Configuration
=============

There are many ways to configure BIND9. Some of the most common
configurations are a caching nameserver, primary master, and as a
secondary master.

-  When configured as a caching nameserver BIND9 will find the answer to
   name queries and remember the answer when the domain is queried
   again.

-  As a primary master server BIND9 reads the data for a zone from a
   file on it's host and is authoritative for that zone.

-  In a secondary master configuration BIND9 gets the zone data from
   another nameserver authoritative for the zone.

Overview
--------

The DNS configuration files are stored in the ``/etc/bind`` directory.
The primary configuration file is ``/etc/bind/named.conf``.

The *include* line specifies the filename which contains the DNS
options. The *directory* line in the ``/etc/bind/named.conf.options``
file tells DNS where to look for files. All files BIND uses will be
relative to this directory.

The file named ``/etc/bind/db.root`` describes the root nameservers in
the world. The servers change over time, so the ``/etc/bind/db.root``
file must be maintained now and then. This is usually done as updates to
the bind9 package. The *zone* section defines a master server, and it is
stored in a file mentioned in the *file* option.

It is possible to configure the same server to be a caching name server,
primary master, and secondary master. A server can be the Start of
Authority (SOA) for one zone, while providing secondary service for
another zone. All the while providing caching services for hosts on the
local LAN.

Caching Nameserver
------------------

The default configuration is setup to act as a caching server. All that
is required is simply adding the IP Addresses of your ISP's DNS servers.
Simply uncomment and edit the following in
``/etc/bind/named.conf.options``:

::

    forwarders {
                    1.2.3.4;
                    5.6.7.8;
               };

    **Note**

    Replace *1.2.3.4* and *5.6.7.8* with the IP Adresses of actual
    nameservers.

Now restart the DNS server, to enable the new configuration. From a
terminal prompt:

::

See ? for information on testing a caching DNS server.

Primary Master
--------------

In this section BIND9 will be configured as the Primary Master for the
domain *example.com*. Simply replace *example.com* with your FQDN (Fully
Qualified Domain Name).

Forward Zone File
~~~~~~~~~~~~~~~~~

To add a DNS zone to BIND9, turning BIND9 into a Primary Master server,
the first step is to edit ``/etc/bind/named.conf.local``:

::

    zone "example.com" {
        type master;
            file "/etc/bind/db.example.com";
    };

(Note, if bind will be receiving automatic updates to the file as with
DDNS, then use ``/var/lib/bind/db.example.com`` rather than
``/etc/bind/db.example.com`` both here and in the copy command below.)

Now use an existing zone file as a template to create the
``/etc/bind/db.example.com`` file:

::

Edit the new zone file ``/etc/bind/db.example.com`` change *localhost.*
to the FQDN of your server, leaving the additional "." at the end.
Change *127.0.0.1* to the nameserver's IP Address and *root.localhost*
to a valid email address, but with a "." instead of the usual "@"
symbol, again leaving the "." at the end. Change the comment to indicate
the domain that this file is for.

Create an *A record* for the base domain, *example.com*. Also, create an
*A record* for *ns.example.com*, the name server in this example:

::

    ;
    ; BIND data file for example.com
    ;
    $TTL    604800
    @       IN      SOA     example.com. root.example.com. (
                                  2         ; Serial
                             604800         ; Refresh
                              86400         ; Retry
                            2419200         ; Expire
                             604800 )       ; Negative Cache TTL
            IN      A       192.168.1.10
    ;
    @       IN      NS      ns.example.com.
    @       IN      A       192.168.1.10
    @       IN      AAAA    ::1
    ns      IN      A       192.168.1.10

You must increment the *Serial Number* every time you make changes to
the zone file. If you make multiple changes before restarting BIND9,
simply increment the Serial once.

Now, you can add DNS records to the bottom of the zone file. See ? for
details.

    **Note**

    Many admins like to use the last date edited as the serial of a
    zone, such as *2012010100* which is yyyymmddss (where *ss* is the
    Serial Number)

Once you have made changes to the zone file BIND9 needs to be restarted
for the changes to take effect:

::

Reverse Zone File
~~~~~~~~~~~~~~~~~

Now that the zone is setup and resolving names to IP Adresses a *Reverse
zone* is also required. A Reverse zone allows DNS to resolve an address
to a name.

Edit /etc/bind/named.conf.local and add the following:

::

    zone "1.168.192.in-addr.arpa" {
            type master;
            file "/etc/bind/db.192";
    };

    **Note**

    Replace *1.168.192* with the first three octets of whatever network
    you are using. Also, name the zone file ``/etc/bind/db.192``
    appropriately. It should match the first octet of your network.

Now create the ``/etc/bind/db.192`` file:

::

Next edit ``/etc/bind/db.192`` changing the basically the same options
as ``/etc/bind/db.example.com``:

::

    ;
    ; BIND reverse data file for local 192.168.1.XXX net
    ;
    $TTL    604800
    @       IN      SOA     ns.example.com. root.example.com. (
                                  2         ; Serial
                             604800         ; Refresh
                              86400         ; Retry
                            2419200         ; Expire
                             604800 )       ; Negative Cache TTL
    ;
    @       IN      NS      ns.
    10      IN      PTR     ns.example.com.

The *Serial Number* in the Reverse zone needs to be incremented on each
change as well. For each *A record* you configure in
``/etc/bind/db.example.com``, that is for a different address, you need
to create a *PTR record* in ``/etc/bind/db.192``.

After creating the reverse zone file restart BIND9:

::

Secondary Master
----------------

Once a *Primary Master* has been configured a *Secondary Master* is
needed in order to maintain the availability of the domain should the
Primary become unavailable.

First, on the Primary Master server, the zone transfer needs to be
allowed. Add the *allow-transfer* option to the example Forward and
Reverse zone definitions in ``/etc/bind/named.conf.local``:

::

    zone "example.com" {
            type master;
        file "/etc/bind/db.example.com";
            allow-transfer { 192.168.1.11; };
    };

    zone "1.168.192.in-addr.arpa" {
            type master;
            file "/etc/bind/db.192";
        allow-transfer { 192.168.1.11; };
    };

    **Note**

    Replace *192.168.1.11* with the IP Address of your Secondary
    nameserver.

Restart BIND9 on the Primary Master:

::

Next, on the Secondary Master, install the bind9 package the same way as
on the Primary. Then edit the ``/etc/bind/named.conf.local`` and add the
following declarations for the Forward and Reverse zones:

::

    zone "example.com" {
        type slave;
            file "db.example.com";
            masters { 192.168.1.10; };
    };        
          
    zone "1.168.192.in-addr.arpa" {
        type slave;
            file "db.192";
            masters { 192.168.1.10; };
    };

    **Note**

    Replace *192.168.1.10* with the IP Address of your Primary
    nameserver.

Restart BIND9 on the Secondary Master:

::

In ``/var/log/syslog`` you should see something similar to (some lines
have been split to fit the format of this document):

::

    client 192.168.1.10#39448: received notify for zone '1.168.192.in-addr.arpa'
    zone 1.168.192.in-addr.arpa/IN: Transfer started.
    transfer of '100.18.172.in-addr.arpa/IN' from 192.168.1.10#53:
     connected using 192.168.1.11#37531
    zone 1.168.192.in-addr.arpa/IN: transferred serial 5
    transfer of '100.18.172.in-addr.arpa/IN' from 192.168.1.10#53:
     Transfer completed: 1 messages, 
    6 records, 212 bytes, 0.002 secs (106000 bytes/sec)
    zone 1.168.192.in-addr.arpa/IN: sending notifies (serial 5)

    client 192.168.1.10#20329: received notify for zone 'example.com'
    zone example.com/IN: Transfer started.
    transfer of 'example.com/IN' from 192.168.1.10#53: connected using 192.168.1.11#38577
    zone example.com/IN: transferred serial 5
    transfer of 'example.com/IN' from 192.168.1.10#53: Transfer completed: 1 messages, 
    8 records, 225 bytes, 0.002 secs (112500 bytes/sec)

    **Note**

    Note: A zone is only transferred if the *Serial Number* on the
    Primary is larger than the one on the Secondary. If you want to have
    your Primary Master DNS notifying Secondary DNS Servers of zone
    changes, you can add *also-notify { ipaddress; };* in to
    ``/etc/bind/named.conf.local`` as shown in the example below:

::

    zone "example.com" {
        type master;
        file "/etc/bind/db.example.com";
        allow-transfer { 192.168.1.11; };
        also-notify { 192.168.1.11; }; 
        };

    zone "1.168.192.in-addr.arpa" {
        type master;
        file "/etc/bind/db.192";
        allow-transfer { 192.168.1.11; };
        also-notify { 192.168.1.11; }; 
        };
        

    **Note**

    The default directory for non-authoritative zone files is
    ``/var/cache/bind/``. This directory is also configured in AppArmor
    to allow the named daemon to write to it. For more information on
    AppArmor see ?.

Troubleshooting
===============

This section covers ways to help determine the cause when problems
happen with DNS and BIND9.

Testing
-------

resolv.conf
~~~~~~~~~~~

The first step in testing BIND9 is to add the nameserver's IP Address to
a hosts resolver. The Primary nameserver should be configured as well as
another host to double check things. Refer to ? for details on adding
nameserver addresses to your network clients, and afterwards check that
the file ``/etc/resolv.conf`` contains (for this example):

::

    nameserver  192.168.1.10
    nameserver  192.168.1.11

Nameservers that listen at 127.\* are responsible for adding their own
IP addresses to resolv.conf (using resolvconf). This is done via the
file ``/etc/default/bind9`` by changing the line RESOLVCONF=no to
RESOLVCONF=yes.

    **Note**

    You should also add the IP Address of the Secondary nameserver in
    case the Primary becomes unavailable.

dig
~~~

If you installed the dnsutils package you can test your setup using the
DNS lookup utility dig:

-  After installing BIND9 use dig against the loopback interface to make
   sure it is listening on port 53. From a terminal prompt:

   ::

   You should see lines similar to the following in the command output:

   ::

       ;; Query time: 1 msec
       ;; SERVER: 192.168.1.10#53(192.168.1.10)

-  If you have configured BIND9 as a *Caching* nameserver "dig" an
   outside domain to check the query time:

   ::

   Note the query time toward the end of the command output:

   ::

       ;; Query time: 49 msec

   After a second dig there should be improvement:

   ::

       ;; Query time: 1 msec

ping
~~~~

Now to demonstrate how applications make use of DNS to resolve a host
name use the ping utility to send an ICMP echo request. From a terminal
prompt enter:

::

This tests if the nameserver can resolve the name *ns.example.com* to an
IP Address. The command output should resemble:

::

    PING ns.example.com (192.168.1.10) 56(84) bytes of data.
    64 bytes from 192.168.1.10: icmp_seq=1 ttl=64 time=0.800 ms
    64 bytes from 192.168.1.10: icmp_seq=2 ttl=64 time=0.813 ms

named-checkzone
~~~~~~~~~~~~~~~

A great way to test your zone files is by using the named-checkzone
utility installed with the bind9 package. This utility allows you to
make sure the configuration is correct before restarting BIND9 and
making the changes live.

-  To test our example Forward zone file enter the following from a
   command prompt:

   ::

   If everything is configured correctly you should see output similar
   to:

   ::

       zone example.com/IN: loaded serial 6
       OK

-  Similarly, to test the Reverse zone file enter the following:

   ::

   The output should be similar to:

   ::

       zone 1.168.192.in-addr.arpa/IN: loaded serial 3
       OK

    **Note**

    The *Serial Number* of your zone file will probably be different.

Logging
-------

BIND9 has a wide variety of logging configuration options available.
There are two main options. The *channel* option configures where logs
go, and the *category* option determines what information to log.

If no logging option is configured the default option is:

::

    logging {
         category default { default_syslog; default_debug; };
         category unmatched { null; };
    };

This section covers configuring BIND9 to send *debug* messages related
to DNS queries to a separate file.

-  First, we need to configure a channel to specify which file to send
   the messages to. Edit ``/etc/bind/named.conf.local`` and add the
   following:

   ::

       logging {
           channel query.log {      
               file "/var/log/query.log";
               severity debug 3; 
           }; 
       };

-  Next, configure a category to send all DNS queries to the query file:

   ::

       logging {
           channel query.log {      
               file "/var/log/query.log"; 
               severity debug 3; 
           }; 
            
       };

    **Note**

    Note: the *debug* option can be set from 1 to 3. If a level isn't
    specified level 1 is the default.

-  Since the *named daemon* runs as the *bind* user the
   ``/var/log/query.log`` file must be created and the ownership
   changed:

   ::


-  Before named daemon can write to the new log file the AppArmor
   profile must be updated. First, edit
   ``/etc/apparmor.d/usr.sbin.named`` and add:

   ::

       /var/log/query.log w,

   Next, reload the profile:

   ::

   For more information on AppArmor see ?

-  Now restart BIND9 for the changes to take effect:

   ::

You should see the file ``/var/log/query.log`` fill with query
information. This is a simple example of the BIND9 logging options. For
coverage of advanced options see ?.

References
==========

Common Record Types
-------------------

This section covers some of the most common DNS record types.

-  *A* record: This record maps an IP Address to a hostname.

   ::

       www      IN    A      192.168.1.12

-  *CNAME* record: Used to create an alias to an existing A record. You
   cannot create a CNAME record pointing to another CNAME record.

   ::

       web     IN    CNAME  www

-  *MX* record: Used to define where email should be sent to. Must point
   to an A record, not a CNAME.

   ::

               IN    MX  1   mail.example.com.
       mail    IN    A       192.168.1.13

-  *NS* record: Used to define which servers serve copies of a zone. It
   must point to an A record, not a CNAME. This is where Primary and
   Secondary servers are defined.

   ::

               IN    NS     ns.example.com.
               IN    NS     ns2.example.com.
       ns      IN    A      192.168.1.10
       ns2     IN    A      192.168.1.11

More Information
----------------

-  The `BIND9 Server
   HOWTO <https://help.ubuntu.com/community/BIND9ServerHowto>`__ in the
   Ubuntu Wiki has a lot of useful information.

-  The `DNS HOWTO <http://www.tldp.org/HOWTO/DNS-HOWTO.html>`__ at The
   Linux Documentation Project also has lots of information about
   configuring BIND9.

-  `Bind9.net <http://www.bind9.net/>`__ has links to a large collection
   of DNS and BIND9 resources.

-  `DNS and BIND <http://shop.oreilly.com/product/9780596100575.do>`__
   is a popular book now in it's fifth edition. There is now also a `DNS
   and BIND on
   IPv6 <http://shop.oreilly.com/product/0636920020158.do>`__ book.

-  A great place to ask for BIND9 assistance, and get involved with the
   Ubuntu Server community, is the *#ubuntu-server* IRC channel on
   `freenode <http://freenode.net>`__.


