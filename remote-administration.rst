Remote Administration
=====================

There are many ways to remotely administer a Linux server. This chapter
will cover three of the most popular applications OpenSSH, Puppet, and
Zentyal.

OpenSSH Server
==============

Introduction
------------

This section of the Ubuntu SG-TITLE introduces a powerful collection of
tools for the remote control of, and transfer of data between, networked
computers called *OpenSSH*. You will also learn about some of the
configuration settings possible with the OpenSSH server application and
how to change them on your Ubuntu system.

OpenSSH is a freely available version of the Secure Shell (SSH) protocol
family of tools for remotely controlling, or transferring files between,
computers. Traditional tools used to accomplish these functions, such as
telnet or rcp, are insecure and transmit the user's password in
cleartext when used. OpenSSH provides a server daemon and client tools
to facilitate secure, encrypted remote control and file transfer
operations, effectively replacing the legacy tools.

The OpenSSH server component, sshd, listens continuously for client
connections from any of the client tools. When a connection request
occurs, sshd sets up the correct connection depending on the type of
client tool connecting. For example, if the remote computer is
connecting with the ssh client application, the OpenSSH server sets up a
remote control session after authentication. If a remote user connects
to an OpenSSH server with scp, the OpenSSH server daemon initiates a
secure copy of files between the server and client after authentication.
OpenSSH can use many authentication methods, including plain password,
public key, and Kerberos tickets.

Installation
------------

Installation of the OpenSSH client and server applications is simple. To
install the OpenSSH client applications on your Ubuntu system, use this
command at a terminal prompt:

::

To install the OpenSSH server application, and related support files,
use this command at a terminal prompt:

::

The openssh-server package can also be selected to install during the
Server Edition installation process.

Configuration
-------------

You may configure the default behavior of the OpenSSH server
application, sshd, by editing the file ``/etc/ssh/sshd_config``. For
information about the configuration directives used in this file, you
may view the appropriate manual page with the following command, issued
at a terminal prompt:

::

There are many directives in the sshd configuration file controlling
such things as communication settings, and authentication modes. The
following are examples of configuration directives that can be changed
by editing the ``/etc/ssh/sshd_config`` file.

    **Tip**

    Prior to editing the configuration file, you should make a copy of
    the original file and protect it from writing so you will have the
    original settings as a reference and to reuse as necessary.

    Copy the ``/etc/ssh/sshd_config`` file and protect it from writing
    with the following commands, issued at a terminal prompt:

    ::


The following are examples of configuration directives you may change:

-  To set your OpenSSH to listen on TCP port 2222 instead of the default
   TCP port 22, change the Port directive as such:

   Port 2222

-  To have sshd allow public key-based login credentials, simply add or
   modify the line:

   PubkeyAuthentication yes

   If the line is already present, then ensure it is not commented out.

-  To make your OpenSSH server display the contents of the
   ``/etc/issue.net`` file as a pre-login banner, simply add or modify
   the line:

   Banner /etc/issue.net

   In the ``/etc/ssh/sshd_config`` file.

After making changes to the ``/etc/ssh/sshd_config`` file, save the
file, and restart the sshd server application to effect the changes
using the following command at a terminal prompt:

::

    **Warning**

    Many other configuration directives for sshd are available to change
    the server application's behavior to fit your needs. Be advised,
    however, if your only method of access to a server is ssh, and you
    make a mistake in configuring sshd via the ``/etc/ssh/sshd_config``
    file, you may find you are locked out of the server upon restarting
    it. Additionally, if an incorrect configuration directive is
    supplied, the sshd server may refuse to start, so be extra careful
    when editing this file on a remote server.

SSH Keys
--------

SSH *keys* allow authentication between two hosts without the need of a
password. SSH key authentication uses two keys, a *private* key and a
*public* key.

To generate the keys, from a terminal prompt enter:

::

This will generate the keys using the *Digital Signature Algorithm
(DSA)* method. During the process you will be prompted for a password.
Simply hit *Enter* when prompted to create the key.

By default the *public* key is saved in the file ``~/.ssh/id_dsa.pub``,
while ``~/.ssh/id_dsa`` is the *private* key. Now copy the
``id_dsa.pub`` file to the remote host and append it to
``~/.ssh/authorized_keys`` by entering:

::

Finally, double check the permissions on the ``authorized_keys`` file,
only the authenticated user should have read and write permissions. If
the permissions are not correct change them by:

::

You should now be able to SSH to the host without being prompted for a
password.

References
----------

-  `Ubuntu Wiki SSH <https://help.ubuntu.com/community/SSH>`__ page.

-  `OpenSSH Website <http://www.openssh.org/>`__

-  `Advanced OpenSSH Wiki
   Page <https://wiki.ubuntu.com/AdvancedOpenSSH>`__

Puppet
======

Puppet is a cross platform framework enabling system administrators to
perform common tasks using code. The code can do a variety of tasks from
installing new software, to checking file permissions, or updating user
accounts. Puppet is great not only during the initial installation of a
system, but also throughout the system's entire life cycle. In most
circumstances puppet will be used in a client/server configuration.

This section will cover installing and configuring Puppet in a
client/server configuration. This simple example will demonstrate how to
install Apache using Puppet.

Preconfiguration
----------------

Prior to configuring puppet you may want to add a DNS *CNAME* record for
*puppet.example.com*, where *example.com* is your domain. By default
Puppet clients check DNS for puppet.example.com as the puppet server
name, or *Puppet Master*. See ? for more DNS details.

If you do not wish to use DNS, you can add entries to the server and
client ``/etc/hosts`` file. For example, in the Puppet server's
``/etc/hosts`` file add:

::

    127.0.0.1 localhost.localdomain localhost puppet
    192.168.1.17 puppetclient.example.com puppetclient

On each Puppet client, add an entry for the server:

::

    192.168.1.16 puppetmaster.example.com puppetmaster puppet

    **Note**

    Replace the example IP addresses and domain names above with your
    actual server and client addresses and domain names.

Installation
------------

To install Puppet, in a terminal on the *server* enter:

::

On the *client* machine, or machines, enter:

::

Configuration
-------------

Create a folder path for the apache2 class:

::

      

Now setup some resources for apache2. Create a file
``/etc/puppet/modules/apache2/manifests/init.pp`` containing the
following:

::

    class apache2 {
      package { 'apache2':
        ensure => installed,
      }

      service { 'apache2':
        ensure  => true,
        enable  => true,
        require => Package['apache2'],
      }
    }

Next, create a node file ``/etc/puppet/manifests/site.pp`` with:

::

    node 'puppetclient.example.com' {
       include apache2
    }

    **Note**

    Replace *puppetclient.example.com* with your actual Puppet client's
    host name.

The final step for this simple Puppet server is to restart the daemon:

::

Now everything is configured on the Puppet server, it is time to
configure the client.

First, configure the Puppet agent daemon to start. Edit
``/etc/default/puppet``, changing *START* to yes:

::

    START=yes

Then start the service:

::

View the client cert fingerprint

::

Back on the Puppet server, view pending certificate signing requests:

::

On the Puppet server, verify the fingerprint of the client and sign
puppetclient's cert:

::

On the Puppet client, run the puppet agent manually in the foreground.
This step isn't strictly speaking necessary, but it is the best way to
test and debug the puppet service.

::

Check ``/var/log/syslog`` on both hosts for any errors with the
configuration. If all goes well the apache2 package and it's
dependencies will be installed on the Puppet client.

    **Note**

    This example is *very* simple, and does not highlight many of
    Puppet's features and benefits. For more information see ?.

Resources
---------

-  See the `Official Puppet
   Documentation <http://docs.puppetlabs.com/>`__ web site.

-  See the `Puppet forge <http://forge.puppetlabs.com/>`__, online
   repository of puppet modules.

-  Also see `Pro Puppet <http://www.apress.com/9781430230571>`__.

Zentyal
=======

Zentyal is a Linux small business server, that can be configured as a
Gateway, Infrastructure Manager, Unified Threat Manager, Office Server,
Unified Communication Server or a combination of them. All network
services managed by Zentyal are tightly integrated, automating most
tasks. This helps to avoid errors in the network configuration and
administration and allows to save time. Zentyal is open source, released
under the GNU General Public License (GPL) and runs on top of Ubuntu
GNU/Linux.

Zentyal consists of a serie of packages (usually one for each module)
that provide a web interface to configure the different servers or
services. The configuration is stored on a key-value Redis database but
users, groups and domains related configuration is on OpenLDAP. When you
configure any of the available parameters through the web interface,
final configuration files are overwritten using the configuration
templates provided by the modules. The main advantages of using Zentyal
are: unified, graphical user interface to configure all network services
and high, out-of-the-box integration between them.

Installation
------------

Zentyal 2.3 is available on Ubuntu 12.04 Universe repository. The
modules available are:

-  zentyal-core & zentyal-common: the core of the Zentyal interface and
   the common libraries of the framework. Also include the logs and
   events modules that give the administrator an interface to view the
   logs and generate events from them.

-  zentyal-network: manages the configuration of the network. From the
   interfaces (supporting static IP, DHCP, VLAN, bridges or PPPoE), to
   multiple gateways when having more than one Internet connection, load
   balancing and advanced routing, static routes or dynamic DNS.

-  zentyal-objects & zentyal-services: provide an abstraction level for
   network addresses (e.g. LAN instead of 192.168.1.0/24) and ports
   named as services (e.g. HTTP instead of 80/TCP).

-  zentyal-firewall: configures the iptables rules to block forbiden
   connections, NAT and port redirections.

-  zentyal-ntp: installs the NTP daemon to keep server on time and allow
   network clients to synchronize their clocks against the server.

-  zentyal-dhcp: configures ISC DHCP server supporting network ranges,
   static leases and other advanced options like NTP, WINS, dynamic DNS
   updates and network boot with PXE.

-  zentyal-dns: brings ISC Bind9 DNS server into your server for caching
   local queries as a forwarder or as an authoritative server for the
   configured domains. Allows to configure A, CNAME, MX, NS, TXT and SRV
   records.

-  zentyal-ca: integrates the management of a Certification Authority
   within Zentyal so users can use certificates to authenticate against
   the services, like with OpenVPN.

-  zentyal-openvpn: allows to configure multiple VPN servers and clients
   using OpenVPN with dynamic routing configuration using Quagga.

-  zentyal-users: provides an interface to configure and manage users
   and groups on OpenLDAP. Other services on Zentyal are authenticated
   against LDAP having a centralized users and groups management. It is
   also possible to synchronize users, passwords and groups from a
   Microsoft Active Directory domain.

-  zentyal-squid: configures Squid and Dansguardian for speeding up
   browsing thanks to the caching capabilities and content filtering.

-  zentyal-samba: allows Samba configuration and integration with
   existing LDAP. From the same interface you can define password
   policies, create shared resources and assign permissions.

-  zentyal-printers: integrates CUPS with Samba and allows not only to
   configure the printers but also give them permissions based on LDAP
   users and groups.

To install Zentyal, in a terminal on the *server* enter (where
<zentyal-module> is any of the modules from the previous list):

::

    **Note**

    Zentyal publishes one major stable release once a year (in
    September) based on latest Ubuntu LTS release. Stable releases
    always have even minor numbers (e.g. 2.2, 3.0) and beta releases
    have odd minor numbers (e.g. 2.1, 2.3). Ubuntu 12.04 comes with
    Zentyal 2.3 packages. If you want to upgrade to a new stable release
    published after the release of Ubuntu 12.04 you can use `Zentyal
    Team PPA <https://launchpad.net/~zentyal/>`__. Upgrading to newer
    stable releases can provide you minor bugfixes not backported to 2.3
    in Precise and newer features.

    **Tip**

    If you need more information on how to add packages from a PPA see
    `Add a Personal Package Archive
    (PPA) <https://help.ubuntu.com/&distro-rev-short;/ubuntu-help/addremove-ppa.html>`__.

    **Note**

    Not present on Ubuntu Universe repositories, but on `Zentyal Team
    PPA <https://launchpad.net/~zentyal/>`__ you will find these other
    modules:

    -  zentyal-antivirus: integrates ClamAV antivirus with other modules
       like the proxy, file sharing or mailfilter.

    -  zentyal-asterisk: configures Asterisk to provide a simple PBX
       with LDAP based authentication.

    -  zentyal-bwmonitor: allows to monitor bandwith usage of your LAN
       clients.

    -  zentyal-captiveportal: integrates a captive portal with the
       firewall and LDAP users and groups.

    -  zentyal-ebackup: allows to make scheduled backups of your server
       using the popular duplicity backup tool.

    -  zentyal-ftp: configures a FTP server with LDAP based
       authentication.

    -  zentyal-ids: integrates a network intrusion detection system.

    -  zentyal-ipsec: allows to configure IPsec tunnels using OpenSwan.

    -  zentyal-jabber: integrates ejabberd XMPP server with LDAP users
       and groups.

    -  zentyal-thinclients: a LTSP based thin clients solution.

    -  zentyal-mail: a full mail stack including Postfix and Dovecot
       with LDAP backend.

    -  zentyal-mailfilter: configures amavisd with mail stack to filter
       spam and attached virus.

    -  zentyal-monitor: integrates collectd to monitor server
       performance and running services.

    -  zentyal-pptp: configures a PPTP VPN server.

    -  zentyal-radius: integrates FreeRADIUS with LDAP users and groups.

    -  zentyal-software: simple interface to manage installed Zentyal
       modules and system updates.

    -  zentyal-trafficshaping: configures traffic limiting rules to do
       bandwidth throttling and improve latency.

    -  zentyal-usercorner: allows users to edit their own LDAP
       attributes using a web browser.

    -  zentyal-virt: simple interface to create and manage virtual
       machines based on libvirt.

    -  zentyal-webmail: allows to access your mail using the popular
       Roundcube webmail.

    -  zentyal-webserver: configures Apache webserver to host different
       sites on your machine.

    -  zentyal-zarafa: integrates Zarafa groupware suite with Zentyal
       mail stack and LDAP.

First steps
-----------

Any system account belonging to the sudo group is allowed to log into
Zentyal web interface. If you are using the user created during the
installation, this should be in the sudo group by default.

    **Tip**

    If you need to add another user to the sudo group, just execute:

    ::

To access Zentyal web interface, browse into https://localhost/ (or the
IP of your remote server). As Zentyal creates its own self-signed SSL
certificate, you will have to accept a security exception on your
browser.

Once logged in you will see the dashboard with an overview of your
server. To configure any of the features of your installed modules, go
to the different sections on the left menu. When you make any changes,
on the upper right corner appears a red *Save changes* button that you
must click to save all configuration changes. To apply these
configuration changes in your server, the module needs to be enabled
first, you can do so from the *Module Status* entry on the left menu.
Every time you enable a module, a pop-up will appear asking for a
confirmation to perform the necessary actions and changes on your server
and configuration files.

    **Note**

    If you need to customize any configuration file or run certain
    actions (scripts or commands) to configure features not available on
    Zentyal place the custom configuration file templates on
    /etc/zentyal/stubs/<module>/ and the hooks on
    /etc/zentyal/hooks/<module>.<action>.

References
----------

`Zentyal Official Documentation <http://doc.zentyal.org/>`__ page.

See also `Zentyal Community
Documentation <http://trac.zentyal.org/wiki/Documentation>`__ page.

And don't forget to visit the `forum <http://forum.zentyal.org/>`__ for
community support, feedback, feature requests, etc.
