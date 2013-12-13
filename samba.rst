Samba
=====

Computer networks are often comprised of diverse systems, and while
operating a network made up entirely of Ubuntu desktop and server
computers would certainly be fun, some network environments must consist
of both Ubuntu and Microsoft Windows systems working together in
harmony. This section of the UBUNTU SG-TITLE introduces principles and
tools used in configuring your Ubuntu Server for sharing network
resources with Windows computers.

Introduction
============

Successfully networking your Ubuntu system with Windows clients involves
providing and integrating with services common to Windows environments.
Such services assist the sharing of data and information about the
computers and users involved in the network, and may be classified under
three major categories of functionality:

-  **File and Printer Sharing Services**. Using the Server Message Block
   (SMB) protocol to facilitate the sharing of files, folders, volumes,
   and the sharing of printers throughout the network.

-  **Directory Services**. Sharing vital information about the computers
   and users of the network with such technologies as the Lightweight
   Directory Access Protocol (LDAP) and Microsoft Active Directory.

-  **Authentication and Access**. Establishing the identity of a
   computer or user of the network and determining the information the
   computer or user is authorized to access using such principles and
   technologies as file permissions, group policies, and the Kerberos
   authentication service.

Fortunately, your Ubuntu system may provide all such facilities to
Windows clients and share network resources among them. One of the
principal pieces of software your Ubuntu system includes for Windows
networking is the Samba suite of SMB server applications and tools.

This section of the UBUNTU SG-TITLE will introduce some of the common
Samba use cases, and how to install and configure the necessary
packages. Additional detailed documentation and information on Samba can
be found on the `Samba website <http://www.samba.org>`__.

File Server
===========

One of the most common ways to network Ubuntu and Windows computers is
to configure Samba as a File Server. This section covers setting up a
Samba server to share files with Windows clients.

The server will be configured to share files with any client on the
network without prompting for a password. If your environment requires
stricter Access Controls see ?

Installation
------------

The first step is to install the samba package. From a terminal prompt
enter:

::

That's all there is to it; you are now ready to configure Samba to share
files.

Configuration
-------------

The main Samba configuration file is located in ``/etc/samba/smb.conf``.
The default configuration file has a significant amount of comments in
order to document various configuration directives.

    **Note**

    Not all the available options are included in the default
    configuration file. See the ``smb.conf`` man page or the `Samba
    HOWTO
    Collection <http://samba.org/samba/docs/man/Samba-HOWTO-Collection/>`__
    for more details.

First, edit the following key/value pairs in the *[global]* section of
``/etc/samba/smb.conf``:

::

       workgroup = EXAMPLE
       ...
       security = user

The *security* parameter is farther down in the [global] section, and is
commented by default. Also, change *EXAMPLE* to better match your
environment.

Create a new section at the bottom of the file, or uncomment one of the
examples, for the directory to be shared:

::

    [share]
        comment = Ubuntu File Server Share
        path = /srv/samba/share
        browsable = yes
        guest ok = yes
        read only = no
        create mask = 0755

-  *comment:* a short description of the share. Adjust to fit your
   needs.

-  *path:* the path to the directory to share.

   This example uses ``/srv/samba/sharename`` because, according to the
   *Filesystem Hierarchy Standard (FHS)*,
   `/srv <http://www.pathname.com/fhs/pub/fhs-2.3.html#SRVDATAFORSERVICESPROVIDEDBYSYSTEM>`__
   is where site-specific data should be served. Technically Samba
   shares can be placed anywhere on the filesystem as long as the
   permissions are correct, but adhering to standards is recommended.

-  *browsable:* enables Windows clients to browse the shared directory
   using Windows Explorer.

-  *guest ok:* allows clients to connect to the share without supplying
   a password.

-  *read only:* determines if the share is read only or if write
   privileges are granted. Write privileges are allowed only when the
   value is *no*, as is seen in this example. If the value is *yes*,
   then access to the share is read only.

-  *create mask:* determines the permissions new files will have when
   created.

Now that Samba is configured, the directory needs to be created and the
permissions changed. From a terminal enter:

::


    **Note**

    The *-p* switch tells mkdir to create the entire directory tree if
    it doesn't exist.

Finally, restart the samba services to enable the new configuration:

::


    **Warning**

    Once again, the above configuration gives all access to any client
    on the local network. For a more secure configuration see ?.

From a Windows client you should now be able to browse to the Ubuntu
file server and see the shared directory. If your client doesn't show
your share automatically, try to access your server by its IP address,
e.g. \\\\192.168.1.1, in a Windows Explorer window. To check that
everything is working try creating a directory from Windows.

To create additional shares simply create new *[dir]* sections in
``/etc/samba/smb.conf``, and restart *Samba*. Just make sure that the
directory you want to share actually exists and the permissions are
correct.

    **Note**

    The file share named *"[share]"* and the path ``/srv/samba/share``
    are just examples. Adjust the share and path names to fit your
    environment. It is a good idea to name a share after a directory on
    the file system. Another example would be a share name of *[qa]*
    with a path of ``/srv/samba/qa``.

Resources
---------

-  For in depth Samba configurations see the `Samba HOWTO
   Collection <http://samba.org/samba/docs/man/Samba-HOWTO-Collection/>`__

-  The guide is also available in `printed
   format <http://www.amazon.com/exec/obidos/tg/detail/-/0131882228>`__.

-  O'Reilly's `Using
   Samba <http://www.oreilly.com/catalog/9780596007690/>`__ is another
   good reference.

-  The `Ubuntu Wiki Samba <https://help.ubuntu.com/community/Samba>`__
   page.

Print Server
============

Another common use of Samba is to configure it to share printers
installed, either locally or over the network, on an Ubuntu server.
Similar to ? this section will configure Samba to allow any client on
the local network to use the installed printers without prompting for a
username and password.

For a more secure configuration see ?.

Installation
------------

Before installing and configuring Samba it is best to already have a
working CUPS installation. See ? for details.

To install the samba package, from a terminal enter:

::

Configuration
-------------

After installing samba edit ``/etc/samba/smb.conf``. Change the
*workgroup* attribute to what is appropriate for your network, and
change *security* to *user*:

::

       workgroup = EXAMPLE
       ...
       security = user

In the *[printers]* section change the *guest ok* option to *yes*:

::

       browsable = yes
       guest ok = yes

After editing ``smb.conf`` restart Samba:

::


The default Samba configuration will automatically share any printers
installed. Simply install the printer locally on your Windows clients.

Resources
---------

-  For in depth Samba configurations see the `Samba HOWTO
   Collection <http://samba.org/samba/docs/man/Samba-HOWTO-Collection/>`__

-  The guide is also available in `printed
   format <http://www.amazon.com/exec/obidos/tg/detail/-/0131882228>`__.

-  O'Reilly's `Using
   Samba <http://www.oreilly.com/catalog/9780596007690/>`__ is another
   good reference.

-  Also, see the `CUPS Website <http://www.cups.org/>`__ for more
   information on configuring CUPS.

-  The `Ubuntu Wiki Samba <https://help.ubuntu.com/community/Samba>`__
   page.

Securing File and Print Server
==============================

Samba Security Modes
--------------------

There are two security levels available to the Common Internet
Filesystem (CIFS) network protocol *user-level* and *share-level*.
Samba's *security mode* implementation allows more flexibility,
providing four ways of implementing user-level security and one way to
implement share-level:

-  *security = user:* requires clients to supply a username and password
   to connect to shares. Samba user accounts are separate from system
   accounts, but the libpam-smbpass package will sync system users and
   passwords with the Samba user database.

-  *security = domain:* this mode allows the Samba server to appear to
   Windows clients as a Primary Domain Controller (PDC), Backup Domain
   Controller (BDC), or a Domain Member Server (DMS). See ? for further
   information.

-  *security = ADS:* allows the Samba server to join an Active Directory
   domain as a native member. See ? for details.

-  *security = server:* this mode is left over from before Samba could
   become a member server, and due to some security issues should not be
   used. See the `Server
   Security <http://samba.org/samba/docs/man/Samba-HOWTO-Collection/ServerType.html#id349531>`__
   section of the Samba guide for more details.

-  *security = share:* allows clients to connect to shares without
   supplying a username and password.

The security mode you choose will depend on your environment and what
you need the Samba server to accomplish.

Security = User
---------------

This section will reconfigure the Samba file and print server, from ?
and ?, to require authentication.

First, install the libpam-smbpass package which will sync the system
users to the Samba user database:

::

    **Note**

    If you chose the *Samba Server* task during installation
    libpam-smbpass is already installed.

Edit ``/etc/samba/smb.conf``, and in the *[share]* section change:

::

        guest ok = no

Finally, restart Samba for the new settings to take effect:

::


Now when connecting to the shared directories or printers you should be
prompted for a username and password.

    **Note**

    If you choose to map a network drive to the share you can check the
    “Reconnect at Logon” check box, which will require you to only enter
    the username and password once, at least until the password changes.

Share Security
--------------

There are several options available to increase the security for each
individual shared directory. Using the *[share]* example, this section
will cover some common options.

Groups
~~~~~~

Groups define a collection of computers or users which have a common
level of access to particular network resources and offer a level of
granularity in controlling access to such resources. For example, if a
group *qa* is defined and contains the users *freda*, *danika*, and
*rob* and a second group *support* is defined and consists of users
*danika*, *jeremy*, and *vincent* then certain network resources
configured to allow access by the *qa* group will subsequently enable
access by freda, danika, and rob, but not jeremy or vincent. Since the
user *danika* belongs to both the *qa* and *support* groups, she will be
able to access resources configured for access by both groups, whereas
all other users will have only access to resources explicitly allowing
the group they are part of.

By default Samba looks for the local system groups defined in
``/etc/group`` to determine which users belong to which groups. For more
information on adding and removing users from groups see ?.

When defining groups in the Samba configuration file,
``/etc/samba/smb.conf``, the recognized syntax is to preface the group
name with an "@" symbol. For example, if you wished to define a group
named *sysadmin* in a certain section of the ``/etc/samba/smb.conf``,
you would do so by entering the group name as **@sysadmin**.

File Permissions
~~~~~~~~~~~~~~~~

File Permissions define the explicit rights a computer or user has to a
particular directory, file, or set of files. Such permissions may be
defined by editing the ``/etc/samba/smb.conf`` file and specifying the
explicit permissions of a defined file share.

For example, if you have defined a Samba share called *share* and wish
to give *read-only* permissions to the group of users known as *qa*, but
wanted to allow writing to the share by the group called *sysadmin* and
the user named *vincent*, then you could edit the
``/etc/samba/smb.conf`` file, and add the following entries under the
*[share]* entry:

::

        read list = @qa
        write list = @sysadmin, vincent

Another possible Samba permission is to declare *administrative*
permissions to a particular shared resource. Users having administrative
permissions may read, write, or modify any information contained in the
resource the user has been given explicit administrative permissions to.

For example, if you wanted to give the user *melissa* administrative
permissions to the *share* example, you would edit the
``/etc/samba/smb.conf`` file, and add the following line under the
*[share]* entry:

::

        admin users = melissa

After editing ``/etc/samba/smb.conf``, restart Samba for the changes to
take effect:

::


    **Note**

    For the *read list* and *write list* to work the Samba security mode
    must *not* be set to *security = share*

Now that Samba has been configured to limit which groups have access to
the shared directory, the filesystem permissions need to be updated.

Traditional Linux file permissions do not map well to Windows NT Access
Control Lists (ACLs). Fortunately POSIX ACLs are available on Ubuntu
servers providing more fine grained control. For example, to enable ACLs
on ``/srv`` an EXT3 filesystem, edit ``/etc/fstab`` adding the *acl*
option:

::

    UUID=66bcdd2e-8861-4fb0-b7e4-e61c569fe17d /srv  ext3    noatime,relatime,acl 0       1

Then remount the partition:

::

    **Note**

    The above example assumes ``/srv`` on a separate partition. If
    ``/srv``, or wherever you have configured your share path, is part
    of the ``/`` partition a reboot may be required.

To match the Samba configuration above the *sysadmin* group will be
given read, write, and execute permissions to ``/srv/samba/share``, the
*qa* group will be given read and execute permissions, and the files
will be owned by the username *melissa*. Enter the following in a
terminal:

::



    **Note**

    The setfacl command above gives *execute* permissions to all files
    in the ``/srv/samba/share`` directory, which you may or may not
    want.

Now from a Windows client you should notice the new file permissions are
implemented. See the acl and setfacl man pages for more information on
POSIX ACLs.

Samba AppArmor Profile
----------------------

Ubuntu comes with the AppArmor security module, which provides mandatory
access controls. The default AppArmor profile for Samba will need to be
adapted to your configuration. For more details on using AppArmor see ?.

There are default AppArmor profiles for ``/usr/sbin/smbd`` and
``/usr/sbin/nmbd``, the Samba daemon binaries, as part of the
apparmor-profiles packages. To install the package, from a terminal
prompt enter:

::

    **Note**

    This package contains profiles for several other binaries.

By default the profiles for smbd and nmbd are in *complain* mode
allowing Samba to work without modifying the profile, and only logging
errors. To place the smbd profile into *enforce* mode, and have Samba
work as expected, the profile will need to be modified to reflect any
directories that are shared.

Edit ``/etc/apparmor.d/usr.sbin.smbd`` adding information for *[share]*
from the file server example:

::

      /srv/samba/share/ r,
      /srv/samba/share/** rwkix,

Now place the profile into *enforce* and reload it:

::


You should now be able to read, write, and execute files in the shared
directory as normal, and the smbd binary will have access to only the
configured files and directories. Be sure to add entries for each
directory you configure Samba to share. Also, any errors will be logged
to ``/var/log/syslog``.

Resources
---------

-  For in depth Samba configurations see the `Samba HOWTO
   Collection <http://samba.org/samba/docs/man/Samba-HOWTO-Collection/>`__

-  The guide is also available in `printed
   format <http://www.amazon.com/exec/obidos/tg/detail/-/0131882228>`__.

-  O'Reilly's `Using
   Samba <http://www.oreilly.com/catalog/9780596007690/>`__ is also a
   good reference.

-  `Chapter
   18 <http://samba.org/samba/docs/man/Samba-HOWTO-Collection/securing-samba.html>`__
   of the Samba HOWTO Collection is devoted to security.

-  For more information on Samba and ACLs see the `Samba ACLs
   page <http://samba.org/samba/docs/man/Samba-HOWTO-Collection/AccessControls.html#id397568>`__.

-  The `Ubuntu Wiki Samba <https://help.ubuntu.com/community/Samba>`__
   page.

As a Domain Controller
======================

Although it cannot act as an Active Directory Primary Domain Controller
(PDC), a Samba server can be configured to appear as a Windows NT4-style
domain controller. A major advantage of this configuration is the
ability to centralize user and machine credentials. Samba can also use
multiple backends to store the user information.

Primary Domain Controller
-------------------------

This section covers configuring Samba as a Primary Domain Controller
(PDC) using the default smbpasswd backend.

First, install Samba, and libpam-smbpass to sync the user accounts, by
entering the following in a terminal prompt:

::

Next, configure Samba by editing ``/etc/samba/smb.conf``. The *security*
mode should be set to *user*, and the *workgroup* should relate to your
organization:

::

       workgroup = EXAMPLE
       ...
       security = user

In the commented “Domains” section add or uncomment the following (the
last line has been split to fit the format of this document):

::

       domain logons = yes
       logon path = \\%N\%U\profile
       logon drive = H:
       logon home = \\%N\%U
       logon script = logon.cmd
       add machine script = sudo /usr/sbin/useradd -N -g machines -c Machine -d
             /var/lib/samba -s /bin/false %u

    **Note**

    If you wish to not use *Roaming Profiles* leave the *logon home* and
    *logon path* options commented.

-  *domain logons:* provides the netlogon service causing Samba to act
   as a domain controller.

-  *logon path:* places the user's Windows profile into their home
   directory. It is also possible to configure a *[profiles]* share
   placing all profiles under a single directory.

-  *logon drive:* specifies the home directory local path.

-  *logon home:* specifies the home directory location.

-  *logon script:* determines the script to be run locally once a user
   has logged in. The script needs to be placed in the *[netlogon]*
   share.

-  *add machine script:* a script that will automatically create the
   *Machine Trust Account* needed for a workstation to join the domain.

   In this example the *machines* group will need to be created using
   the addgroup utility see ? for details.

Uncomment the *[homes]* share to allow the *logon home* to be mapped:

::

    [homes]
       comment = Home Directories
       browseable = no
       read only = no
       create mask = 0700
       directory mask = 0700
       valid users = %S

When configured as a domain controller a *[netlogon]* share needs to be
configured. To enable the share, uncomment:

::

    [netlogon]
       comment = Network Logon Service
       path = /srv/samba/netlogon
       guest ok = yes
       read only = yes
       share modes = no

    **Note**

    The original *netlogon* share path is ``/home/samba/netlogon``, but
    according to the Filesystem Hierarchy Standard (FHS),
    `/srv <http://www.pathname.com/fhs/pub/fhs-2.3.html#SRVDATAFORSERVICESPROVIDEDBYSYSTEM>`__
    is the correct location for site-specific data provided by the
    system.

Now create the ``netlogon`` directory, and an empty (for now)
``logon.cmd`` script file:

::


You can enter any normal Windows logon script commands in ``logon.cmd``
to customize the client's environment.

Restart Samba to enable the new domain controller:

::


Lastly, there are a few additional commands needed to setup the
appropriate rights.

With *root* being disabled by default, in order to join a workstation to
the domain, a system group needs to be mapped to the Windows *Domain
Admins* group. Using the net utility, from a terminal enter:

::

    **Note**

    Change *sysadmin* to whichever group you prefer. Also, the user used
    to join the domain needs to be a member of the *sysadmin* group, as
    well as a member of the system *admin* group. The *admin* group
    allows sudo use.

    If the user does not have Samba credentials yet, you can add them
    with the smbpasswd utility, change the *sysadmin* username
    appropriately:

    ::

Also, rights need to be explicitly provided to the *Domain Admins* group
to allow the *add machine script* (and other admin functions) to work.
This is achieved by executing:

::

You should now be able to join Windows clients to the Domain in the same
manner as joining them to an NT4 domain running on a Windows server.

Backup Domain Controller
------------------------

With a Primary Domain Controller (PDC) on the network it is best to have
a Backup Domain Controller (BDC) as well. This will allow clients to
authenticate in case the PDC becomes unavailable.

When configuring Samba as a BDC you need a way to sync account
information with the PDC. There are multiple ways of accomplishing this
scp, rsync, or by using LDAP as the *passdb backend*.

Using LDAP is the most robust way to sync account information, because
both domain controllers can use the same information in real time.
However, setting up a LDAP server may be overly complicated for a small
number of user and computer accounts. See ? for details.

First, install samba and libpam-smbpass. From a terminal enter:

::

Now, edit ``/etc/samba/smb.conf`` and uncomment the following in the
*[global]*:

::

       workgroup = EXAMPLE
       ...
       security = user

In the commented *Domains* uncomment or add:

::

       domain logons = yes
       domain master = no

Make sure a user has rights to read the files in ``/var/lib/samba``. For
example, to allow users in the *admin* group to scp the files, enter:

::

Next, sync the user accounts, using scp to copy the ``/var/lib/samba``
directory from the PDC:

::

    **Note**

    Replace *username* with a valid username and *pdc* with the hostname
    or IP Address of your actual PDC.

Finally, restart samba:

::


You can test that your Backup Domain controller is working by stopping
the Samba daemon on the PDC, then trying to login to a Windows client
joined to the domain.

Another thing to keep in mind is if you have configured the *logon home*
option as a directory on the PDC, and the PDC becomes unavailable,
access to the user's *Home* drive will also be unavailable. For this
reason it is best to configure the *logon home* to reside on a separate
file server from the PDC and BDC.

Resources
---------

-  For in depth Samba configurations see the `Samba HOWTO
   Collection <http://samba.org/samba/docs/man/Samba-HOWTO-Collection/>`__

-  The guide is also available in `printed
   format <http://www.amazon.com/exec/obidos/tg/detail/-/0131882228>`__.

-  O'Reilly's `Using
   Samba <http://www.oreilly.com/catalog/9780596007690/>`__ is also a
   good reference.

-  `Chapter
   4 <http://samba.org/samba/docs/man/Samba-HOWTO-Collection/samba-pdc.html>`__
   of the Samba HOWTO Collection explains setting up a Primary Domain
   Controller.

-  `Chapter
   5 <http://us3.samba.org/samba/docs/man/Samba-HOWTO-Collection/samba-bdc.html>`__
   of the Samba HOWTO Collection explains setting up a Backup Domain
   Controller.

-  The `Ubuntu Wiki Samba <https://help.ubuntu.com/community/Samba>`__
   page.

Active Directory Integration
============================

Accessing a Samba Share
-----------------------

Another, use for Samba is to integrate into an existing Windows network.
Once part of an Active Directory domain, Samba can provide file and
print services to AD users.

The simplest way to join an AD domain is to use Likewise-open. For
detailed instructions see the `Likewise Open
documentation <http://www.beyondtrust.com/Technical-Support/Downloads/files/pbiso/Manuals/ubuntu-active-directory.html>`__.

Once part of the Active Directory domain, enter the following command in
the terminal prompt:

::

Next, edit ``/etc/samba/smb.conf`` changing:

::

       workgroup = EXAMPLE
       ...
       security = ads
       realm = EXAMPLE.COM
       ...
       idmap backend = lwopen
       idmap uid = 50-9999999999
       idmap gid = 50-9999999999

Restart samba for the new settings to take effect:

::


You should now be able to access any Samba shares from a Windows client.
However, be sure to give the appropriate AD users or groups access to
the share directory. See ? for more details.

Accessing a Windows Share
-------------------------

Now that the Samba server is part of the Active Directory domain you can
access any Windows server shares:

-  To mount a Windows file share enter the following in a terminal
   prompt:

   ::

   It is also possible to access shares on computers not part of an AD
   domain, but a username and password will need to be provided.

-  To mount the share during boot place an entry in ``/etc/fstab``, for
   example:

   ::

       //192.168.0.5/share /mnt/windows cifs auto,username=steve,password=secret,rw 0        0

-  Another way to copy files from a Windows server is to use the
   smbclient utility. To list the files in a Windows share:

   ::

-  To copy a file from the share, enter:

   ::

   This will copy the ``file.txt`` into the current directory.

-  And to copy a file to the share:

   ::

   This will copy the ``/etc/hosts`` to
   ``//fs01.example.com/share/hosts``.

-  The *-c* option used above allows you to execute the smbclient
   command all at once. This is useful for scripting and minor file
   operations. To enter the *smb: \\>* prompt, a FTP like prompt where
   you can execute normal file and directory commands, simply execute:

   ::

    **Note**

    Replace all instances of *fs01.example.com/share*,
    *//192.168.0.5/share*, *username=steve,password=secret*, and
    *file.txt* with your server's IP, hostname, share name, file name,
    and an actual username and password with rights to the share.

Resources
---------

For more smbclient options see the man page: ``man smbclient``, also
available
`online <http://manpages.ubuntu.com/manpages/&distro-short-codename;/en/man1/smbclient.1.html>`__.

The mount.cifs `man
page <http://manpages.ubuntu.com/manpages/&distro-short-codename;/en/man8/mount.cifs.8.html>`__
is also useful for more detailed information.

The `Ubuntu Wiki Samba <https://help.ubuntu.com/community/Samba>`__
page.
