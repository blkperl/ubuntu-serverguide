Monitoring
==========

Overview
========

The monitoring of essential servers and services is an important part of
system administration. Most network services are monitored for
performance, availability, or both. This section will cover installation
and configuration of Nagios for availability monitoring, and Munin for
performance monitoring.

The examples in this section will use two servers with hostnames
*server01* and *server02*. *Server01* will be configured with Nagios to
monitor services on itself and *server02*. Server01 will also be setup
with the munin package to gather information from the network. Using the
munin-node package, *server02* will be configured to send information to
*server01*.

Hopefully these simple examples will allow you to monitor additional
servers and services on your network.

Nagios
======

Installation
------------

First, on *server01* install the nagios package. In a terminal enter:

::

You will be asked to enter a password for the *nagiosadmin* user. The
user's credentials are stored in ``/etc/nagios3/htpasswd.users``. To
change the *nagiosadmin* password, or add additional users to the Nagios
CGI scripts, use the htpasswd that is part of the apache2-utils package.

For example, to change the password for the *nagiosadmin* user enter:

::

To add a user:

::

Next, on *server02* install the nagios-nrpe-server package. From a
terminal on server02 enter:

::

    **Note**

    NRPE allows you to execute local checks on remote hosts. There are
    other ways of accomplishing this through other Nagios plugins as
    well as other checks.

Configuration Overview
----------------------

There are a couple of directories containing Nagios configuration and
check files.

-  ``/etc/nagios3``: contains configuration files for the operation of
   the nagios daemon, CGI files, hosts, etc.

-  ``/etc/nagios-plugins``: houses configuration files for the service
   checks.

-  ``/etc/nagios``: on the remote host contains the nagios-nrpe-server
   configuration files.

-  ``/usr/lib/nagios/plugins/``: where the check binaries are stored. To
   see the options of a check use the *-h* option.

   For example: ``/usr/lib/nagios/plugins/check_dhcp -h``

There are a plethora of checks Nagios can be configured to execute for
any given host. For this example Nagios will be configured to check disk
space, DNS, and a MySQL hostgroup. The DNS check will be on *server02*,
and the MySQL hostgroup will include both *server01* and *server02*.

    **Note**

    See ? for details on setting up Apache, ? for DNS, and ? for MySQL.

Additionally, there are some terms that once explained will hopefully
make understanding Nagios configuration easier:

-  *Host*: a server, workstation, network device, etc that is being
   monitored.

-  *Host Group*: a group of similar hosts. For example, you could group
   all web servers, file server, etc.

-  *Service*: the service being monitored on the host. Such as HTTP,
   DNS, NFS, etc.

-  *Service Group*: allows you to group multiple services together. This
   is useful for grouping multiple HTTP for example.

-  *Contact*: person to be notified when an event takes place. Nagios
   can be configured to send emails, SMS messages, etc.

By default Nagios is configured to check HTTP, disk space, SSH, current
users, processes, and load on the *localhost*. Nagios will also ping
check the *gateway*.

Large Nagios installations can be quite complex to configure. It is
usually best to start small, one or two hosts, get things configured the
way you like then expand.

Configuration
-------------

-  First, create a *host* configuration file for *server02*. Unless
   otherwise specified, run all these commands on *server01*. In a
   terminal enter:

   ::

       **Note**

       In the above and following command examples, replace
       *"server01"*, *"server02"* *172.18.100.100*, and *172.18.100.101*
       with the host names and IP addresses of your servers.

   Next, edit ``/etc/nagios3/conf.d/server02.cfg``:

   ::

       define host{
               use                     generic-host  ; Name of host template to use
               host_name               server02
               alias                   Server 02
               address                 172.18.100.101
       }

       # check DNS service.
       define service {
               use                             generic-service
               host_name                       server02
               service_description             DNS
               check_command                   check_dns!172.18.100.101
       }

   Restart the nagios daemon to enable the new configuration:

   ::

-  Now add a service definition for the MySQL check by adding the
   following to ``/etc/nagios3/conf.d/services_nagios2.cfg``:

   ::

       # check MySQL servers.
       define service {
               hostgroup_name        mysql-servers
               service_description   MySQL
               check_command         check_mysql_cmdlinecred!nagios!secret!$HOSTADDRESS
               use                   generic-service
               notification_interval 0 ; set > 0 if you want to be renotified
       }

   A *mysql-servers* hostgroup now needs to be defined. Edit
   ``/etc/nagios3/conf.d/hostgroups_nagios2.cfg`` adding:

   ::

       # MySQL hostgroup.
       define hostgroup {
               hostgroup_name  mysql-servers
                       alias           MySQL servers
                       members         localhost, server02
               }

   The Nagios check needs to authenticate to MySQL. To add a *nagios*
   user to MySQL enter:

   ::

       **Note**

       The *nagios* user will need to be added all hosts in the
       *mysql-servers* hostgroup.

   Restart nagios to start checking the MySQL servers.

   ::

-  Lastly configure NRPE to check the disk space on *server02*.

   On *server01* add the service check to
   ``/etc/nagios3/conf.d/server02.cfg``:

   ::

       # NRPE disk check.
       define service {
               use                     generic-service
               host_name               server02
               service_description     nrpe-disk
               check_command           check_nrpe_1arg!check_all_disks!172.18.100.101
       }

   Now on *server02* edit ``/etc/nagios/nrpe.cfg`` changing:

   ::

       allowed_hosts=172.18.100.100

   And below in the command definition area add:

   ::

       command[check_all_disks]=/usr/lib/nagios/plugins/check_disk -w 20% -c 10% -e

   Finally, restart nagios-nrpe-server:

   ::

   Also, on *server01* restart nagios:

   ::

You should now be able to see the host and service checks in the Nagios
CGI files. To access them point a browser to http://server01/nagios3.
You will then be prompted for the *nagiosadmin* username and password.

References
----------

This section has just scratched the surface of Nagios' features. The
nagios-plugins-extra and nagios-snmp-plugins contain many more service
checks.

-  For more information see `Nagios <http://www.nagios.org/>`__ website.

-  Specifically the `Online
   Documentation <http://nagios.sourceforge.net/docs/3_0/>`__ site.

-  There is also a list of
   `books <http://www.nagios.org/propaganda/books/>`__ related to Nagios
   and network monitoring:

-  The `Nagios Ubuntu
   Wiki <https://help.ubuntu.com/community/Nagios3>`__ page also has
   more details.

Munin
=====

Installation
------------

Before installing Munin on *server01* apache2 will need to be installed.
The default configuration is fine for running a munin server. For more
information see ?.

First, on *server01* install munin. In a terminal enter:

::

Now on *server02* install the munin-node package:

::

Configuration
-------------

On *server01* edit the ``/etc/munin/munin.conf`` adding the IP address
for *server02*:

::

    ## First our "normal" host.
    [server02]
           address 172.18.100.101

    **Note**

    Replace *server02* and *172.18.100.101* with the actual hostname and
    IP address for your server.

Next, configure munin-node on *server02*. Edit
``/etc/munin/munin-node.conf`` to allow access by *server01*:

::

    allow ^172\.18\.100\.100$

    **Note**

    Replace *^172\\.18\\.100\\.100$* with IP address for your munin
    server.

Now restart munin-node on *server02* for the changes to take effect:

::

Finally, in a browser go to *http://server01/munin*, and you should see
links to nice graphs displaying information from the standard
*munin-plugins* for disk, network, processes, and system.

    **Note**

    Since this is a new install it may take some time for the graphs to
    display anything useful.

Additional Plugins
------------------

The munin-plugins-extra package contains performance checks additional
services such as DNS, DHCP, Samba, etc. To install the package, from a
terminal enter:

::

Be sure to install the package on both the server and node machines.

References
----------

-  See the `Munin <http://munin.projects.linpro.no/>`__ website for more
   details.

-  Specifically the `Munin
   Documentation <http://munin.projects.linpro.no/wiki/Documentation>`__
   page includes information on additional plugins, writing plugins,
   etc.

-  Also, there is a book in German by Open Source Press: `Munin
   Graphisches Netzwerk- und
   System-Monitoring <https://www.opensourcepress.de/index.php?26&backPID=178&tt_products=152>`__.


