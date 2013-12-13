File Servers
============

If you have more than one computer on a single network. At some point
you will probably need to share files between them. In this section we
cover installing and configuring FTP, NFS, and CUPS.

FTP Server
==========

File Transfer Protocol (FTP) is a TCP protocol for downloading files
between computers. In the past, it has also been used for uploading but,
as that method does not use encryption, user credentials as well as data
transferred in the clear and are easily intercepted. So if you are here
looking for a way to upload and download files securely, see the section
on OpenSSH in ? instead.

FTP works on a client/server model. The server component is called an
*FTP daemon*. It continuously listens for FTP requests from remote
clients. When a request is received, it manages the login and sets up
the connection. For the duration of the session it executes any of
commands sent by the FTP client.

Access to an FTP server can be managed in two ways:

-  Anonymous

-  Authenticated

In the Anonymous mode, remote clients can access the FTP server by using
the default user account called "anonymous" or "ftp" and sending an
email address as the password. In the Authenticated mode a user must
have an account and a password. This latter choice is very insecure and
should not be used except in special circumstances. If you are looking
to transfer files securely see SFTP in the section on OpenSSH-Server.
User access to the FTP server directories and files is dependent on the
permissions defined for the account used at login. As a general rule,
the FTP daemon will hide the root directory of the FTP server and change
it to the FTP Home directory. This hides the rest of the file system
from remote sessions.

vsftpd - FTP Server Installation
--------------------------------

vsftpd is an FTP daemon available in Ubuntu. It is easy to install, set
up, and maintain. To install vsftpd you can run the following command:

::

Anonymous FTP Configuration
---------------------------

By default vsftpd is *not* configured to allow anonymous download. If
you wish to enable anonymous download edit ``/etc/vsftpd.conf`` by
changing:

::

    anonymous_enable=Yes

During installation a *ftp* user is created with a home directory of
``/srv/ftp``. This is the default FTP directory.

If you wish to change this location, to ``/srv/files/ftp`` for example,
simply create a directory in another location and change the *ftp*
user's home directory:

::


     

After making the change restart vsftpd:

::

Finally, copy any files and directories you would like to make available
through anonymous FTP to ``/srv/files/ftp``, or ``/srv/ftp`` if you wish
to use the default.

User Authenticated FTP Configuration
------------------------------------

By default vsftpd is configured to authenticate system users and allow
them to download files. If you want users to be able to upload files,
edit ``/etc/vsftpd.conf``:

::

    write_enable=YES

Now restart vsftpd:

::

Now when system users login to FTP they will start in their *home*
directories where they can download, upload, create directories, etc.

Similarly, by default, anonymous users are not allowed to upload files
to FTP server. To change this setting, you should uncomment the
following line, and restart vsftpd:

::

    anon_upload_enable=YES

    **Warning**

    Enabling anonymous FTP upload can be an extreme security risk. It is
    best to not enable anonymous upload on servers accessed directly
    from the Internet.

The configuration file consists of many configuration parameters. The
information about each parameter is available in the configuration file.
Alternatively, you can refer to the man page, ``man 5 vsftpd.conf`` for
details of each parameter.

Securing FTP
------------

There are options in ``/etc/vsftpd.conf`` to help make vsftpd more
secure. For example users can be limited to their home directories by
uncommenting:

::

    chroot_local_user=YES

You can also limit a specific list of users to just their home
directories:

::

    chroot_list_enable=YES
    chroot_list_file=/etc/vsftpd.chroot_list

After uncommenting the above options, create a
``/etc/vsftpd.chroot_list`` containing a list of users one per line.
Then restart vsftpd:

::

Also, the ``/etc/ftpusers`` file is a list of users that are
*disallowed* FTP access. The default list includes root, daemon, nobody,
etc. To disable FTP access for additional users simply add them to the
list.

FTP can also be encrypted using *FTPS*. Different from *SFTP*, *FTPS* is
FTP over Secure Socket Layer (SSL). *SFTP* is a FTP like session over an
encrypted *SSH* connection. A major difference is that users of SFTP
need to have a *shell* account on the system, instead of a *nologin*
shell. Providing all users with a shell may not be ideal for some
environments, such as a shared web host. However, it is possible to
restrict such accounts to only SFTP and disable shell interaction. See
the section on OpenSSH-Server for more.

To configure *FTPS*, edit ``/etc/vsftpd.conf`` and at the bottom add:

::

    ssl_enable=Yes

Also, notice the certificate and key related options:

::

    rsa_cert_file=/etc/ssl/certs/ssl-cert-snakeoil.pem
    rsa_private_key_file=/etc/ssl/private/ssl-cert-snakeoil.key

By default these options are set to the certificate and key provided by
the ssl-cert package. In a production environment these should be
replaced with a certificate and key generated for the specific host. For
more information on certificates see ?.

Now restart vsftpd, and non-anonymous users will be forced to use
*FTPS*:

::

To allow users with a shell of ``/usr/sbin/nologin`` access to FTP, but
have no shell access, edit ``/etc/shells`` adding the *nologin* shell:

::

    # /etc/shells: valid login shells
    /bin/csh
    /bin/sh
    /usr/bin/es
    /usr/bin/ksh
    /bin/ksh
    /usr/bin/rc
    /usr/bin/tcsh
    /bin/tcsh
    /usr/bin/esh
    /bin/dash
    /bin/bash
    /bin/rbash
    /usr/bin/screen
    /usr/sbin/nologin

This is necessary because, by default vsftpd uses PAM for
authentication, and the ``/etc/pam.d/vsftpd`` configuration file
contains:

::

    auth    required        pam_shells.so

The *shells* PAM module restricts access to shells listed in the
``/etc/shells`` file.

Most popular FTP clients can be configured to connect using FTPS. The
lftp command line FTP client has the ability to use FTPS as well.

References
----------

-  See the `vsftpd
   website <http://vsftpd.beasts.org/vsftpd_conf.html>`__ for more
   information.

-  For detailed ``/etc/vsftpd.conf`` options see the `vsftpd.conf man
   page <http://manpages.ubuntu.com/manpages/&distro-short-codename;/en/man5/vsftpd.conf.5.html>`__.

Network File System (NFS)
=========================

NFS allows a system to share directories and files with others over a
network. By using NFS, users and programs can access files on remote
systems almost as if they were local files.

Some of the most notable benefits that NFS can provide are:

-  Local workstations use less disk space because commonly used data can
   be stored on a single machine and still remain accessible to others
   over the network.

-  There is no need for users to have separate home directories on every
   network machine. Home directories could be set up on the NFS server
   and made available throughout the network.

-  Storage devices such as floppy disks, CDROM drives, and USB Thumb
   drives can be used by other machines on the network. This may reduce
   the number of removable media drives throughout the network.

Installation
------------

At a terminal prompt enter the following command to install the NFS
Server:

::

Configuration
-------------

You can configure the directories to be exported by adding them to the
``/etc/exports`` file. For example:

::

    /ubuntu  *(ro,sync,no_root_squash)
    /home    *(rw,sync,no_root_squash)

You can replace \* with one of the hostname formats. Make the hostname
declaration as specific as possible so unwanted systems cannot access
the NFS mount.

To start the NFS server, you can run the following command at a terminal
prompt:

::

NFS Client Configuration
------------------------

Use the mount command to mount a shared NFS directory from another
machine, by typing a command line similar to the following at a terminal
prompt:

::

    **Warning**

    The mount point directory ``/local/ubuntu`` must exist. There should
    be no files or subdirectories in the ``/local/ubuntu`` directory.

An alternate way to mount an NFS share from another machine is to add a
line to the ``/etc/fstab`` file. The line must state the hostname of the
NFS server, the directory on the server being exported, and the
directory on the local machine where the NFS share is to be mounted.

The general syntax for the line in ``/etc/fstab`` file is as follows:

::

    example.hostname.com:/ubuntu /local/ubuntu nfs rsize=8192,wsize=8192,timeo=14,intr

If you have trouble mounting an NFS share, make sure the nfs-common
package is installed on your client. To install nfs-common enter the
following command at the terminal prompt:

::

References
----------

`Linux NFS faq <http://nfs.sourceforge.net/>`__

`Ubuntu Wiki NFS Howto <https://help.ubuntu.com/community/NFSv4Howto>`__

iSCSI Initiator
===============

*iSCSI* (Internet Small Computer System Interface) is a protocol that
allows SCSI commands to be transmitted over a network. Typically iSCSI
is implemented in a SAN (Storage Area Network) to allow servers to
access a large store of hard drive space. The iSCSI protocol refers to
clients as *initiators* and iSCSI servers as *targets*.

Ubuntu Server can be configured as both an iSCSI initiator and a target.
This guide provides commands and configuration options to setup an iSCSI
initiator. It is assumed that you already have an iSCSI target on your
local network and have the appropriate rights to connect to it. The
instructions for setting up a target vary greatly between hardware
providers, so consult your vendor documentation to configure your
specific iSCSI target.

iSCSI Initiator Install
-----------------------

To configure Ubuntu Server as an iSCSI initiator install the open-iscsi
package. In a terminal enter:

::

iSCSI Initiator Configuration
-----------------------------

Once the open-iscsi package is installed, edit
``/etc/iscsi/iscsid.conf`` changing the following:

::

    node.startup = automatic

You can check which targets are available by using the iscsiadm utility.
Enter the following in a terminal:

::

-  *-m:* determines the mode that iscsiadm executes in.

-  *-t:* specifies the type of discovery.

-  *-p:* option indicates the target IP address.

    **Note**

    Change example *192.168.0.10* to the target IP address on your
    network.

If the target is available you should see output similar to the
following:

::

    **Note**

    The *iqn* number and IP address above will vary depending on your
    hardware.

You should now be able to connect to the iSCSI target, and depending on
your target setup you may have to enter user credentials. Login to the
iSCSI node:

::

Check to make sure that the new disk has been detected using dmesg:

::


In the output above *sdb* is the new iSCSI disk. Remember this is just
an example; the output you see on your screen will vary.

Next, create a partition, format the file system, and mount the new
iSCSI disk. In a terminal enter:

::





    **Note**

    The above commands are from inside the fdisk utility; see
    ``man fdisk`` for more detailed instructions. Also, the cfdisk
    utility is sometimes more user friendly.

Now format the file system and mount it to ``/srv`` as an example:

::


Finally, add an entry to ``/etc/fstab`` to mount the iSCSI drive during
boot:

::

    /dev/sdb1       /srv        ext4    defaults,auto,_netdev 0 0

It is a good idea to make sure everything is working as expected by
rebooting the server.

References
----------

`Open-iSCSI Website <http://www.open-iscsi.org/>`__

`Debian Open-iSCSI page <http://wiki.debian.org/SAN/iSCSI/open-iscsi>`__

CUPS - Print Server
===================

The primary mechanism for Ubuntu printing and print services is the
**Common UNIX Printing System** (CUPS). This printing system is a freely
available, portable printing layer which has become the new standard for
printing in most Linux distributions.

CUPS manages print jobs and queues and provides network printing using
the standard Internet Printing Protocol (IPP), while offering support
for a very large range of printers, from dot-matrix to laser and many in
between. CUPS also supports PostScript Printer Description (PPD) and
auto-detection of network printers, and features a simple web-based
configuration and administration tool.

Installation
------------

To install CUPS on your Ubuntu computer, simply use sudo with the
apt-get command and give the packages to install as the first parameter.
A complete CUPS install has many package dependencies, but they may all
be specified on the same command line. Enter the following at a terminal
prompt to install CUPS:

::

Upon authenticating with your user password, the packages should be
downloaded and installed without error. Upon the conclusion of
installation, the CUPS server will be started automatically.

For troubleshooting purposes, you can access CUPS server errors via the
error log file at: ``/var/log/cups/error_log``. If the error log does
not show enough information to troubleshoot any problems you encounter,
the verbosity of the CUPS log can be increased by changing the
**LogLevel** directive in the configuration file (discussed below) to
"debug" or even "debug2", which logs everything, from the default of
"info". If you make this change, remember to change it back once you've
solved your problem, to prevent the log file from becoming overly large.

Configuration
-------------

The Common UNIX Printing System server's behavior is configured through
the directives contained in the file ``/etc/cups/cupsd.conf``. The CUPS
configuration file follows the same syntax as the primary configuration
file for the Apache HTTP server, so users familiar with editing Apache's
configuration file should feel at ease when editing the CUPS
configuration file. Some examples of settings you may wish to change
initially will be presented here.

    **Tip**

    Prior to editing the configuration file, you should make a copy of
    the original file and protect it from writing, so you will have the
    original settings as a reference, and to reuse as necessary.

    Copy the ``/etc/cups/cupsd.conf`` file and protect it from writing
    with the following commands, issued at a terminal prompt:

::


-  **ServerAdmin**: To configure the email address of the designated
   administrator of the CUPS server, simply edit the
   ``/etc/cups/cupsd.conf`` configuration file with your preferred text
   editor, and add or modify the *ServerAdmin* line accordingly. For
   example, if you are the Administrator for the CUPS server, and your
   e-mail address is 'bjoy@somebigco.com', then you would modify the
   ServerAdmin line to appear as such:

   ::

       ServerAdmin bjoy@somebigco.com

-  **Listen**: By default on Ubuntu, the CUPS server installation
   listens only on the loopback interface at IP address *127.0.0.1*. In
   order to instruct the CUPS server to listen on an actual network
   adapter's IP address, you must specify either a hostname, the IP
   address, or optionally, an IP address/port pairing via the addition
   of a Listen directive. For example, if your CUPS server resides on a
   local network at the IP address *192.168.10.250* and you'd like to
   make it accessible to the other systems on this subnetwork, you would
   edit the ``/etc/cups/cupsd.conf`` and add a Listen directive, as
   such:

   ::

       Listen 127.0.0.1:631           # existing loopback Listen
       Listen /var/run/cups/cups.sock # existing socket Listen
       Listen 192.168.10.250:631      # Listen on the LAN interface, Port 631 (IPP)

   In the example above, you may comment out or remove the reference to
   the Loopback address (127.0.0.1) if you do not wish cupsd to listen
   on that interface, but would rather have it only listen on the
   Ethernet interfaces of the Local Area Network (LAN). To enable
   listening for all network interfaces for which a certain hostname is
   bound, including the Loopback, you could create a Listen entry for
   the hostname *socrates* as such:

   ::

       Listen socrates:631  # Listen on all interfaces for the hostname 'socrates'

   or by omitting the Listen directive and using *Port* instead, as in:

   ::

       Port 631  # Listen on port 631 on all interfaces

For more examples of configuration directives in the CUPS server
configuration file, view the associated system manual page by entering
the following command at a terminal prompt:

::

    **Note**

    Whenever you make changes to the ``/etc/cups/cupsd.conf``
    configuration file, you'll need to restart the CUPS server by typing
    the following command at a terminal prompt:

::

Web Interface
-------------

    **Tip**

    CUPS can be configured and monitored using a web interface, which by
    default is available at http://localhost:631/admin. The web
    interface can be used to perform all printer management tasks.

In order to perform administrative tasks via the web interface, you must
either have the root account enabled on your server, or authenticate as
a user in the *lpadmin* group. For security reasons, CUPS won't
authenticate a user that doesn't have a password.

To add a user to the *lpadmin* group, run at the terminal prompt:

::

Further documentation is available in the *Documentation/Help* tab of
the web interface.

References
----------

`CUPS Website <http://www.cups.org/>`__

`Debian Open-iSCSI page <http://wiki.debian.org/SAN/iSCSI/open-iscsi>`__
