Virtualization
==============

Virtualization is being adopted in many different environments and
situations. If you are a developer, virtualization can provide you with
a contained environment where you can safely do almost any sort of
development safe from messing up your main working environment. If you
are a systems administrator, you can use virtualization to more easily
separate your services and move them around based on demand.

The default virtualization technology supported in Ubuntu is KVM. KVM
requires virtualization extensions built into Intel and AMD hardware.
Xen is also supported on Ubuntu. Xen can take advantage of
virtualization extensions, when available, but can also be used on
hardware without virtualization extensions. Qemu is another popular
solution for hardware without virtualization extensions.

libvirt
=======

The libvirt library is used to interface with different virtualization
technologies. Before getting started with libvirt it is best to make
sure your hardware supports the necessary virtualization extensions for
KVM. Enter the following from a terminal prompt:

::

A message will be printed informing you if your CPU *does* or *does not*
support hardware virtualization.

    **Note**

    On many computers with processors supporting hardware assisted
    virtualization, it is necessary to activate an option in the BIOS to
    enable it.

Virtual Networking
------------------

There are a few different ways to allow a virtual machine access to the
external network. The default virtual network configuration includes
*bridging* and *iptables* rules implementing *usermode* networking,
which uses the SLIRP protocol. Traffic is NATed through the host
interface to the outside network.

To enable external hosts to directly access services on virtual machines
a different type of *bridge* than the default needs to be configured.
This allows the virtual interfaces to connect to the outside network
through the physical interface, making them appear as normal hosts to
the rest of the network. For information on setting up a bridge see ?.

Installation
------------

To install the necessary packages, from a terminal prompt enter:

::

After installing libvirt-bin, the user used to manage virtual machines
will need to be added to the *libvirtd* group. Doing so will grant the
user access to the advanced networking options.

In a terminal enter:

::

    **Note**

    If the user chosen is the current user, you will need to log out and
    back in for the new group membership to take effect.

You are now ready to install a *Guest* operating system. Installing a
virtual machine follows the same process as installing the operating
system directly on the hardware. You either need a way to automate the
installation, or a keyboard and monitor will need to be attached to the
physical machine.

In the case of virtual machines a Graphical User Interface (GUI) is
analogous to using a physical keyboard and mouse. Instead of installing
a GUI the virt-viewer application can be used to connect to a virtual
machine's console using VNC. See ? for more information.

There are several ways to automate the Ubuntu installation process, for
example using preseeds, kickstart, etc. Refer to the `Ubuntu
Installation
Guide <https://help.ubuntu.com/&distro-rev-short;/installation-guide/>`__
for details.

Yet another way to install an Ubuntu virtual machine is to use
ubuntu-vm-builder. ubuntu-vm-builder allows you to setup advanced
partitions, execute custom post-install scripts, etc. For details see ?

Libvirt can also be configured work with Xen. For details, see the Xen
Ubuntu community page referenced below.

virt-install
------------

virt-install is part of the virtinst package. To install it, from a
terminal prompt enter:

::

There are several options available when using virt-install. For
example:

::

-  *-n web\_devel:* the name of the new virtual machine will be
   *web\_devel* in this example.

-  *-r 256:* specifies the amount of memory the virtual machine will use
   in megabytes.

-  *--disk path=/var/lib/libvirt/images/web\_devel.img,size=4:*
   indicates the path to the virtual disk which can be a file,
   partition, or logical volume. In this example a file named
   ``web_devel.img`` in the /var/lib/libvirt/images/ directory, with a
   size of 4 gigabytes, and using *virtio* for the disk bus.

-  *-c ubuntu-DISTRO-REV-SHORT-server-i386.iso:* file to be used as a
   virtual CDROM. The file can be either an ISO file or the path to the
   host's CDROM device.

-  *--network* provides details related to the VM's network interface.
   Here the *default* network is used, and the interface model is
   configured for *virtio*.

-  *--graphics vnc,listen=0.0.0.0:* exports the guest's virtual console
   using VNC and on all host interfaces. Typically servers have no GUI,
   so another GUI based computer on the Local Area Network (LAN) can
   connect via VNC to complete the installation.

-  *--noautoconsole:* will not automatically connect to the virtual
   machine's console.

-  *-v:* creates a fully virtualized guest.

After launching virt-install you can connect to the virtual machine's
console either locally using a GUI (if your server has a GUI), or via a
remote VNC client from a GUI based computer.

virt-clone
----------

The virt-clone application can be used to copy one virtual machine to
another. For example:

::

     

-  *-o:* original virtual machine.

-  *-n:* name of the new virtual machine.

-  *-f:* path to the file, logical volume, or partition to be used by
   the new virtual machine.

-  *--connect:* specifies which hypervisor to connect to.

Also, use *-d* or *--debug* option to help troubleshoot problems with
virt-clone.

    **Note**

    Replace *web\_devel* and *database\_devel* with appropriate virtual
    machine names.

Virtual Machine Management
--------------------------

virsh
~~~~~

There are several utilities available to manage virtual machines and
libvirt. The virsh utility can be used from the command line. Some
examples:

-  To list running virtual machines:

   ::

-  To start a virtual machine:

   ::

-  Similarly, to start a virtual machine at boot:

   ::

-  Reboot a virtual machine with:

   ::

-  The *state* of virtual machines can be saved to a file in order to be
   restored later. The following will save the virtual machine state
   into a file named according to the date:

   ::

   Once saved the virtual machine will no longer be running.

-  A saved virtual machine can be restored using:

   ::

-  To shutdown a virtual machine do:

   ::

-  A CDROM device can be mounted in a virtual machine by entering:

   ::

    **Note**

    In the above examples replace *web\_devel* with the appropriate
    virtual machine name, and ``web_devel-022708.state`` with a
    descriptive file name.

Virtual Machine Manager
~~~~~~~~~~~~~~~~~~~~~~~

The virt-manager package contains a graphical utility to manage local
and remote virtual machines. To install virt-manager enter:

::

Since virt-manager requires a Graphical User Interface (GUI) environment
it is recommended to be installed on a workstation or test machine
instead of a production server. To connect to the local libvirt service
enter:

::

You can connect to the libvirt service running on another host by
entering the following in a terminal prompt:

::

    **Note**

    The above example assumes that SSH connectivity between the
    management system and virtnode1.mydomain.com has already been
    configured, and uses SSH keys for authentication. SSH *keys* are
    needed because libvirt sends the password prompt to another process.
    For details on configuring SSH see ?

Virtual Machine Viewer
----------------------

The virt-viewer application allows you to connect to a virtual machine's
console. virt-viewer does require a Graphical User Interface (GUI) to
interface with the virtual machine.

To install virt-viewer from a terminal enter:

::

Once a virtual machine is installed and running you can connect to the
virtual machine's console by using:

::

Similar to virt-manager, virt-viewer can connect to a remote host using
*SSH* with key authentication, as well:

::

Be sure to replace *web\_devel* with the appropriate virtual machine
name.

If configured to use a *bridged* network interface you can also setup
SSH access to the virtual machine. See ? and ? for more details.

Resources
---------

-  See the `KVM <http://www.linux-kvm.org/>`__ home page for more
   details.

-  For more information on libvirt see the `libvirt home
   page <http://libvirt.org/>`__

-  The `Virtual Machine Manager <http://virt-manager.et.redhat.com/>`__
   site has more information on virt-manager development.

-  Also, stop by the *#ubuntu-virt* IRC channel on
   `freenode <http://freenode.net/>`__ to discuss virtualization
   technology in Ubuntu.

-  Another good resource is the `Ubuntu Wiki
   KVM <https://help.ubuntu.com/community/KVM>`__ page.

-  For information on Xen, including using Xen with libvirt, please see
   the `Ubuntu Wiki Xen <https://help.ubuntu.com/community/Xen>`__ page.

Cloud images and vmbuilder
==========================

Introduction
------------

With Ubuntu being on of the most used operating systems on most of the
cloud platforms, the availability of stable and secure cloud images has
become very important. Starting with 12.04 the utilization of cloud
images outside of a cloud infrastructure has been improved. It is now
possible to use those images to create a virtual machine without the
need of a complete installation.

Creating virtual machines using cloud images
--------------------------------------------

Cloud images for the supported versions of Ubuntu are available at the
following URL :

-  http://cloud-images.ubuntu.com/

When used in conjunction with a tool called cloud-localds which is part
of the cloud-utils package starting with Ubuntu 12.10 those images can
be used to create a ready to use virtual machine. The following
instructions should give you access to a working virtual machine.

Required packages
~~~~~~~~~~~~~~~~~

The following packages will be required in order to use cloud images as
virtual machines :

-  kvm

-  cloud-utils

-  genisoimage

Get the Ubuntu Cloud Image
~~~~~~~~~~~~~~~~~~~~~~~~~~

The Ubuntu Cloud Image can be downloaded from the Internet by various
means. This example shows how to easily download the 12.04 Precise image
using **wget** :

::

    wget -O my_new_vm.img.dist http://cloud-images.ubuntu.com/server/releases\
    /12.04/release/ubuntu-12.04-server-cloudimg-amd64-disk1.img

Create the user-data file
~~~~~~~~~~~~~~~~~~~~~~~~~

The user-data file contains configuration elements that will be provided
to the cloud image and applied at the first boot of the virtual machine
using cloud-init. The first three elements, **password**, **chpasswd**
and **ssh\_pwauth** are mandatory. You should add an ssh key that you
have created beforehand using ssh-keygen otherwise you will not be able
to connect remotely to your virtual machine.

Use the following command to create the my-user-data file that will
contain your user specific data :

::

    $ cat > my-user-data <<EOF
    #cloud-config
    password: passw0rd
    chpasswd: { expire: False }
    ssh_pwauth: True
    ssh_authorized_keys:
     - ssh-rsa {insert your own ssh public key here}
    EOF

Convert the cloud-image to Qemu format
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The qemu-img command is not strictly necessary:

-  The *convert* command option converts the compressed qcow2 disk image
   as downloaded to an uncompressed version. If you don't do this the
   image will still boot, but reads will undergo decompression.
   Executing the following conversion command will improve the
   performance of your virtual machines.

Use the following command to prepare your file to be used as a virtual
machine disk:

::

    $ qemu-img convert -O qcow2 my_new_vm.img.dist my_new_vm.img

create the disk with NoCloud data on it
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

This action will create a second disk image that will be provided to the
virtual machine as a second disk. The cloud-init initialization process
will fetch this data and configure the virtual machine appropriately

::

    $ cloud-localds my-seed.img my-user-data

Create the XML domain definition file
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

You will need to tailor the following XML domain definition file to your
need in order to create the libvirt domain. If the files that you have
generated are in /home/ubuntu, the template can be used as is.

Use the following command to create the template file :

::

    $ cat > my_new_vm.xml <<EOF
    <domain type='kvm'>
      <name>my_new_vm</name>
      <memory unit='MiB'>1024</memory>
      <currentMemory unit='MiB'>1024</currentMemory>
      <vcpu placement='static'>1</vcpu>
      <os>
        <type arch='x86_64' machine='pc-1.2'>hvm</type>
        <boot dev='hd'/>
        <bootmenu enable='no'/>
      </os>
      <features>
        <acpi/>
        <apic/>
        <pae/>
      </features>
      <clock offset='utc'/>
      <on_poweroff>destroy</on_poweroff>
      <on_reboot>restart</on_reboot>
      <on_crash>restart</on_crash>
      <devices>
        <emulator>/usr/bin/kvm</emulator>
        <disk type='file' device='disk'>
          <driver name='qemu' type='qcow2'/>
          <source file='/home/ubuntu/my_new_vm.img'/>
          <target dev='vda' bus='virtio'/>
          <address type='pci' domain='0x0000' bus='0x00' slot='0x05' function='0x0'/>
        </disk>
        <disk type='file' device='disk'>
          <driver name='qemu' type='raw'/>
          <source file='/home/ubuntu/my-seed.img'/>
          <target dev='hda' bus='ide'/>
          <address type='drive' controller='0' bus='0' target='0' unit='0'/>
        </disk>
        <interface type='network'>
          <source network='default'/>
          <model type='virtio'/>
          <address type='pci' domain='0x0000' bus='0x00' slot='0x03' function='0x0'/>
        </interface>
        <serial type='pty'>
          <target port='0'/>
        </serial>
        <console type='pty'>
          <target type='serial' port='0'/>
        </console>
        <graphics type='vnc' port='-1' autoport='yes'/>
      </devices>
    </domain>
    EOF

Create the VM using libvirt
~~~~~~~~~~~~~~~~~~~~~~~~~~~

The last few commands remaining are standard libvirt commands used to
define and start your virtual machine :

::

    $ virsh define my_new_vm.xml
    $ virsh start my_new_vm
    $ virsh console my_new_vm

If everything goes as planned, you should see the boot sequence appear
in your console session. After the normal boot sequence you will see
something similar to the following :

::

    cloud-init start-local running: Wed, 10 Apr 2013 12:30:25 +0000. up 1.67 seconds
    no instance data found in start-local
    ci-info: lo    : 1 127.0.0.1       255.0.0.0       .
    ci-info: eth0  : 1  255.255.255.0   52:54:00:c2:fd:e1
    ci-info: route-0: 0.0.0.0         192.168.122.1   0.0.0.0         eth0   UG
    ci-info: route-1: 192.168.122.0   0.0.0.0         255.255.255.0   eth0   U
    : Wed, 10 Apr 2013 12:30:30 +0000. up 6.30 seconds
    found data source: DataSourceNoCloud [seed=/dev/sda]

This section can be particularly useful to identify the IP address of
the virtual machine that you have just started. The cloud-init sequence
will continue, creating the SSH information. It should indicates proper
completion by the following line :

::

    cloud-init boot finished at Wed, 10 Apr 2013 12:30:35 +0000. Up 10.93 seconds

Your new virtual machine is now available. You can exit out of the virsh
console command using <Ctrl>]

You can now connect to your virtual machine using the ssh key that you
have created previously :

::

    $ ssh -i $HOME/.ssh/id_rsa ubuntu@192.168.122.113

Vmbuilder
---------

Vmbuilder is now maintained by the community as it is no longer used to
generate the cloud images. It can still be used as described and it
should let you create functioning virtual machines

What is vmbuilder
~~~~~~~~~~~~~~~~~

Vmbuilder will fetch the various package and build a virtual machine
tailored for your needs in about a minute. vmbuilder is a script that
automates the process of creating a ready to use Linux based VM. The
currently supported hypervisors are KVM and Xen.

You can pass command line options to add extra packages, remove
packages, choose which version of Ubuntu, which mirror etc. On recent
hardware with plenty of RAM, tmpdir in ``/dev/shm`` or using a tmpfs,
and a local mirror, you can bootstrap a VM in less than a minute.

First introduced as a shell script in Ubuntu 8.04 LTS, ubuntu-vm-builder
started with little emphasis as a hack to help developers test their new
code in a virtual machine without having to restart from scratch each
time. As a few Ubuntu administrators started to notice this script, a
few of them went on improving it and adapting it for so many use case
that Soren Hansen (the author of the script and Ubuntu virtualization
specialist, not the golf player) decided to rewrite it from scratch for
Intrepid as a python script with a few new design goals:

-  Develop it so that it can be reused by other distributions.

-  Use a plugin mechanisms for all virtualization interactions so that
   others can easily add logic for other virtualization environments.

-  Provide an easy to maintain web interface as an option to the command
   line interface.

But the general principles and commands remain the same.

Initial Setup
~~~~~~~~~~~~~

It is assumed that you have installed and configured libvirt and KVM
locally on the machine you are using. For details on how to perform
this, please refer to:

-  ?

-  The `KVM <https://help.ubuntu.com/community/KVM>`__ Wiki page.

We also assume that you know how to use a text based text editor such as
nano or vi. If you have not used any of them before, you can get an
overview of the various text editors available by reading the
`PowerUsersTextEditors <https://help.ubuntu.com/community/PowerUsersTextEditors>`__
page. This tutorial has been done on KVM, but the general principle
should remain on other virtualization technologies.

Install vmbuilder
~~~~~~~~~~~~~~~~~

The name of the package that we need to install is python-vm-builder. In
a terminal prompt enter:

::

    **Note**

    If you are running Hardy, you can still perform most of this using
    the older version of the package named ubuntu-vm-builder, there are
    only a few changes to the syntax of the tool.

Defining Your Virtual Machine
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Defining a virtual machine with Ubuntu's vmbuilder is quite simple, but
here are a few thing to consider:

-  If you plan on shipping a virtual appliance, do not assume that the
   end-user will know how to extend disk size to fit their need, so
   either plan for a large virtual disk to allow for your appliance to
   grow, or explain fairly well in your documentation how to allocate
   more space. It might actually be a good idea to store data on some
   separate external storage.

-  Given that RAM is much easier to allocate in a VM, RAM size should be
   set to whatever you think is a safe minimum for your appliance.

The vmbuilder command has 2 main parameters: the *virtualization
technology (hypervisor)* and the targeted *distribution*. Optional
parameters are quite numerous and can be found using the following
command:

::

Base Parameters
~~~~~~~~~~~~~~~

As this example is based on KVM and Ubuntu DISTRO-REV (DISTRO-VERSION),
and we are likely to rebuild the same virtual machine multiple time,
we'll invoke vmbuilder with the following first parameters:

::

The *--suite* defines the Ubuntu release, the *--flavour* specifies that
we want to use the virtual kernel (that's the one used to build a JeOS
image), the *--arch* tells that we want to use a 32 bit machine, the
*-o* tells vmbuilder to overwrite the previous version of the VM and the
*--libvirt* tells to inform the local virtualization environment to add
the resulting VM to the list of available machines.

Notes:

-  Because of the nature of operations performed by vmbuilder, it needs
   to have root privilege, hence the use of sudo.

-  If your virtual machine needs to use more than 3Gb of ram, you should
   build a 64 bit machine (--arch amd64).

-  Until Ubuntu 8.10, the virtual kernel was only built for 32 bit
   architecture, so if you want to define an amd64 machine on Hardy, you
   should use *--flavour* server instead.

Installation Parameters
~~~~~~~~~~~~~~~~~~~~~~~

Assigning a fixed IP address
^^^^^^^^^^^^^^^^^^^^^^^^^^^^

As a virtual appliance that may be deployed on various very different
networks, it is very difficult to know what the actual network will look
like. In order to simplify configuration, it is a good idea to take an
approach similar to what network hardware vendors usually do, namely
assigning an initial fixed IP address to the appliance in a private
class network that you will provide in your documentation. An address in
the range 192.168.0.0/255 is usually a good choice.

To do this we'll use the following parameters:

-  *--ip ADDRESS*: IP address in dotted form (defaults to dhcp if not
   specified)

-  *--hostname NAME*: Set NAME as the hostname of the guest.

-  *--mask VALUE*: IP mask in dotted form (default: 255.255.255.0)

-  *--net VALUE*: IP net address (default: X.X.X.0)

-  *--bcast VALUE*: IP broadcast (default: X.X.X.255)

-  *--gw ADDRESS*: Gateway address (default: X.X.X.1)

-  *--dns ADDRESS*: Name server address (default: X.X.X.1)

We assume for now that default values are good enough, so the resulting
invocation becomes:

::

Bridging
''''''''

Because our appliance will be likely to need to be accessed by remote
hosts, we need to configure libvirt so that the appliance uses bridge
networking. To do this add the *--bridge* option to the command:

::

    **Note**

    You will need to have previously setup a bridge interface, see ? for
    more information. Also, if the interface name is different change
    *br0* to the actual bridge interface.

Partitioning
^^^^^^^^^^^^

Partitioning of the virtual appliance will have to take into
consideration what you are planning to do with is. Because most
appliances want to have a separate storage for data, having a separate
``/var`` would make sense.

In order to do this vmbuilder provides us with *--part*:

::

    --part PATH
      Allows you to specify a partition table in a partition file, located at PATH. Each
      line of the partition file should specify (root first):
          mountpoint size
      where  size  is  in megabytes. You can have up to 4 virtual disks, a new disk starts
      on a line with '---'.  ie :
          root 1000
          /opt 1000
          swap 256
          ---
          /var 2000
          /log 1500

In our case we will define a text file name ``vmbuilder.partition``
which will contain the following:

::

    root 8000
    swap 4000
    ---
    /var 20000

    **Note**

    Note that as we are using virtual disk images, the actual sizes that
    we put here are maximum sizes for these volumes.

Our command line now looks like:

::

    **Note**

    Using a "\\" in a command will allow long command strings to wrap to
    the next line.

User and Password
^^^^^^^^^^^^^^^^^

Again setting up a virtual appliance, you will need to provide a default
user and password that is generic so that you can include it in your
documentation. We will see later on in this tutorial how we will provide
some security by defining a script that will be run the first time a
user actually logs in the appliance, that will, among other things, ask
him to change his password. In this example I will use *'user'* as my
user name, and *'default'* as the password.

To do this we use the following optional parameters:

-  *--user USERNAME:* Sets the name of the user to be added. Default:
   ubuntu.

-  *--name FULLNAME:* Sets the full name of the user to be added.
   Default: Ubuntu.

-  *--pass PASSWORD:* Sets the password for the user. Default: ubuntu.

Our resulting command line becomes:

::

Installing Required Packages
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

In this example we will be installing a package (Limesurvey) that
accesses a MySQL database and has a web interface. We will therefore
require our OS to provide us with:

-  Apache

-  PHP

-  MySQL

-  OpenSSH Server

-  Limesurvey (as an example application that we have packaged)

This is done using vmbuilder by specifying the --addpkg option multiple
times:

::

    --addpkg PKG
      Install PKG into the guest (can be specfied multiple times)

However, due to the way vmbuilder operates, packages that have to ask
questions to the user during the post install phase are not supported
and should instead be installed while interactivity can occur. This is
the case of Limesurvey, which we will have to install later, once the
user logs in.

Other packages that ask simple debconf question, such as mysql-server
asking to set a password, the package can be installed immediately, but
we will have to reconfigure it the first time the user logs in.

If some packages that we need to install are not in main, we need to
enable the additional repositories using --comp and --ppa:

::

    --components COMP1,COMP2,...,COMPN
               A comma separated list of distro components to include (e.g. main,universe).
               This defaults to "main"
    --ppa=PPA  Add ppa belonging to PPA to the vm's sources.list.

Limesurvey not being part of the archive at the moment, we'll specify
it's PPA (personal package archive) address so that it is added to the
VM ``/etc/apt/source.list``, so we add the following options to the
command line:

::

Speed Considerations
^^^^^^^^^^^^^^^^^^^^

Package Caching
'''''''''''''''

When vmbuilder creates builds your system, it has to go fetch each one
of the packages that composes it over the network to one of the official
repositories, which, depending on your internet connection speed and the
load of the mirror, can have a big impact on the actual build time. In
order to reduce this, it is recommended to either have a local
repository (which can be created using apt-mirror) or using a caching
proxy such as apt-proxy. The later option being much simpler to
implement and requiring less disk space, it is the one we will pick in
this tutorial. To install it, simply type:

::

Once this is complete, your (empty) proxy is ready for use on
http://mirroraddress:9999 and will find ubuntu repository under /ubuntu.
For vmbuilder to use it, we'll have to use the *--mirror* option:

::

    --mirror=URL  Use Ubuntu mirror at URL instead of the default, which
                  is http://archive.ubuntu.com/ubuntu for official
                  arches and http://ports.ubuntu.com/ubuntu-ports
                  otherwise

So we add to the command line:

::

    **Note**

    The mirror address specified here will also be used in the
    ``/etc/apt/sources.list`` of the newly created guest, so it is
    useful to specify here an address that can be resolved by the guest
    or to plan on reseting this address later on.

Install a Local Mirror
^^^^^^^^^^^^^^^^^^^^^^

If we are in a larger environment, it may make sense to setup a local
mirror of the Ubuntu repositories. The package apt-mirror provides you
with a script that will handle the mirroring for you. You should plan on
having about 20 gigabyte of free space per supported release and
architecture.

By default, apt-mirror uses the configuration file in
``/etc/apt/mirror.list``. As it is set up, it will replicate only the
architecture of the local machine. If you would like to support other
architectures on your mirror, simply duplicate the lines starting with
"deb", replacing the deb keyword by /deb-{arch} where arch can be i386,
amd64, etc... For example, on an amd64 machine, to have the i386
archives as well, you will have (some lines have been split to fit the
format of this document):

::

    deb  http://archive.ubuntu.com/ubuntu DISTRO-SHORT-CODENAME main restricted universe multiverse 
    /deb-i386  http://archive.ubuntu.com/ubuntu DISTRO-SHORT-CODENAME main restricted universe multiverse

    deb  http://archive.ubuntu.com/ubuntu DISTRO-SHORT-CODENAME-updates main restricted universe multiverse 
    /deb-i386  http://archive.ubuntu.com/ubuntu DISTRO-SHORT-CODENAME-updates main
     restricted universe multiverse 

    deb http://archive.ubuntu.com/ubuntu/ DISTRO-SHORT-CODENAME-backports main restricted universe multiverse 
    /deb-i386  http://archive.ubuntu.com/ubuntu DISTRO-SHORT-CODENAME-backports main
     restricted universe multiverse 

    deb http://security.ubuntu.com/ubuntu DISTRO-SHORT-CODENAME-security main restricted universe multiverse 
    /deb-i386  http://security.ubuntu.com/ubuntu DISTRO-SHORT-CODENAME-security main
     restricted universe multiverse 

    deb http://archive.ubuntu.com/ubuntu DISTRO-SHORT-CODENAME main/debian-installer
     restricted/debian-installer universe/debian-installer multiverse/debian-installer 
    /deb-i386 http://archive.ubuntu.com/ubuntu DISTRO-SHORT-CODENAME main/debian-installer
     restricted/debian-installer universe/debian-installer multiverse/debian-installer 

Notice that the source packages are not mirrored as they are seldom used
compared to the binaries and they do take a lot more space, but they can
be easily added to the list.

Once the mirror has finished replicating (and this can be quite long),
you need to configure Apache so that your mirror files (in
``/var/spool/apt-mirror`` if you did not change the default), are
published by your Apache server. For more information on Apache see ?.

Package the Application
~~~~~~~~~~~~~~~~~~~~~~~

Two option are available to us:

-  The recommended method to do so is to make a *Debian* package. Since
   this is outside of the scope of this tutorial, we will not perform
   this here and invite the reader to read the documentation on how to
   do this in the `Ubuntu Packaging
   Guide <https://wiki.ubuntu.com/PackagingGuide>`__. In this case it is
   also a good idea to setup a repository for your package so that
   updates can be conveniently pulled from it. See the `Debian
   Administration <http://www.debian-administration.org/articles/286>`__
   article for a tutorial on this.

-  Manually install the application under ``/opt`` as recommended by the
   `FHS guidelines <http://www.pathname.com/fhs/>`__.

In our case we'll use Limesurvey as example web application for which we
wish to provide a virtual appliance. As noted before, we've made a
version of the package available in a PPA (Personal Package Archive).

Useful Additions
^^^^^^^^^^^^^^^^

Configuring Automatic Updates
'''''''''''''''''''''''''''''

To have your system be configured to update itself on a regular basis,
we will just install unattended-upgrades, so we add the following option
to our command line:

::

As we have put our application package in a PPA, the process will update
not only the system, but also the application each time we update the
version in the PPA.

ACPI Event Handling
'''''''''''''''''''

For your virtual machine to be able to handle restart and shutdown
events it is being sent, it is a good idea to install the acpid package
as well. To do this we just add the following option:

::

Final Command
~~~~~~~~~~~~~

Here is the command with all the options discussed above:

::

Resources
---------

If you are interested in learning more, have questions or suggestions,
please contact the Ubuntu Server Team at:

-  IRC: #ubuntu-server on freenode

-  Mailing list: `ubuntu-server at
   lists.ubuntu.com <https://lists.ubuntu.com/mailman/listinfo/ubuntu-server>`__

-  Also, see the `JeOSVMBuilder Ubuntu
   Wiki <https://help.ubuntu.com/community/JeOSVMBuilder>`__ page.

Ubuntu Cloud
============

Cloud computing is a computing model that allows vast pools of resources
to be allocated on-demand. These resources such as storage, computing
power, network and software are abstracted and delivered as a service
over the Internet anywhere, anytime. These services are billed per time
consumed similar to the ones used by public services such as
electricity, water and telephony. Ubuntu Cloud Infrastructure uses
OpenStack open source software to help build highly scalable, cloud
computing for both public and private clouds.

Overview
--------

This tutorial covers the OpenStack installation from the Ubuntu 12.10
Server Edition CD, and assumes a basic network topology, with a single
system serving as the "all-in-one cloud infrastructure".Due to the
tutorial's simplicity, the instructions as-is are not intended to set up
production servers although it allows you to have a POC (proof of
concept) of the Ubuntu Cloud using OpenStack.

Prerequisites
-------------

To deploy a minimal Ubuntu Cloud infrastructure, you'll need at least:

-  One dedicated system.

-  Two network address ranges (private network and public network).

-  Make sure the host in question supports VT ( Virtualization
   Technology ) since we will be using KVM as the virtualization
   technology. Other hypervisors are also supported such as QEMU, UML,
   Vmware ESX/ESXi and XEN. LXC (Linux Containers) is also supported
   through libvirt.

   Check if your system supports kvm issuing ``sudo kvm-ok`` in a linux
   terminal.

The ``"Minimum Topology"`` recommended for production use is using three
nodes - One master server running nova services (except compute) and two
servers running nova-compute. This setup is not redundant and the master
server is a SPoF (Single Point of Failure).

Preconfiguring the network
--------------------------

Before we start installing OpenStack we need to make sure we have
bridging support installed, a MySQL database, and a central time server
(ntp). This will assure that we have instantiated machines and hosts in
sync.

In this example the "private network" will be in the 10.0.0.0/24 range
on eth1. All the internal communication between instances will happen
there while the "public network" will be in the 10.153.107.0/29 range on
eth0.

Install bridging support
~~~~~~~~~~~~~~~~~~~~~~~~

::

Install and configure NTP
~~~~~~~~~~~~~~~~~~~~~~~~~

::

Add these two lines at the end of the ``/etc/ntp.conf`` file.

::

    server 127.127.1.0
    fudge 127.127.1.0 stratum 10

Restart ntp service

::

Install and configure MySQL
~~~~~~~~~~~~~~~~~~~~~~~~~~~

::

Create a database and mysql user for OpenStack

::


     

The line continuation character "\\" implies that you must include the
subsequent line as part of the current command.

Install OpenStack Compute (Nova)
--------------------------------

``OpenStack Compute (Nova)`` is a cloud computing fabric controller (the
main part of an IaaS system). It is written in Python, using the
Eventlet and Twisted frameworks, and relies on the standard AMQP
messaging protocol, and SQLAlchemy for data store access.

Install OpenStack Nova components

::


Restart libvirt-bin just to make sure libvirtd is aware of ebtables.

::

Install RabbitMQ - Advanced Message Queuing Protocol (AMQP)

::

Edit ``/etc/nova/nova.conf`` and add the following:

::

    # Nova config FlatDHCPManager
    --sql_connection=mysql://novauser:novapassword@localhost/nova
    --flat_injected=true
    --network_manager=nova.network.manager.FlatDHCPManager
    --fixed_range=10.0.0.0/24
    --floating_range=10.153.107.72/29
    --flat_network_dhcp_start=10.0.0.2
    --flat_network_bridge=br100
    --flat_interface=eth1
    --public_interface=eth0

Restart OpenStack services

::


::


Migrate Nova database from sqlite db to MySQL db. It may take a while.

::

Define a specific private network where all your Instances will run.
This will be used in the network of fixed Ips set inside ``nova.conf ``.

::


Define a specific public network and allocate 6 (usable) Floating Public
IP addresses for use with the instances starting from 10.153.107.72.

::

Create a user (user1), a project (project1), download credentials and
source its configuration file.

::






Verify the OpenStack Compute installation by typing:

::


If nova services don't show up correctly restart OpenStack services as
described previously. For more information please refer to the
troubleshooting section on this guide.

Install Imaging Service (Glance)
--------------------------------

Nova uses Glance service to manage Operating System images that it needs
for bringing up instances. Glance can use several types of storage
backends such as filestore, s3 etc. Glance has two components -
*glance-api and glance-registry*. These can be controlled using the
concerned upstart service jobs. For this specific case we will be using
mysql as a storage backend.

Install Glance

::

Create a database and user for glance

::



Edit the file /etc/glance/glance-registry.conf and edit the line which
contains the option "sql\_connection =" to this:

::

    sql_connection = mysql://glanceuser:glancepassword@localhost/glance

Remove the sqlite database

::

Restart glance-registry after making changes to
/etc/glance/glance-registry.conf. The MySQL database will be
automatically populated.

::

If you find issues take a look at the log file in
/var/log/glance/api.log and /var/log/glance/registry.log.

Running Instances
-----------------

Before you can instantiate images, you first need to setup user
credentials. Once this first step is achieved you also need to upload
images that you want to run in the cloud. Once you have these images
uploaded to the cloud you will be able to run and connect to them. Here
are the steps you should follow to get OpenStack Nova running instances:

Download, register and publish an Ubuntu cloud image

::


Create a key pair and start an instance

::




Allow icmp (ping) and ssh access to instances

::


Run an instance

::



Assign public address to the instance.

::



You must enter above the instance\_id (ami) and public\_ip\_address
shown above by euca-describe-instances and euca-allocate-address
commands.

Now you should be able to SSH to the instance

::

To terminate instances

::

Install the Storage Infrastructure (Swift)
------------------------------------------

Swift is a highly available, distributed, eventually consistent
object/blob store. It is used by the OpenStack Infrastructure to provide
S3 like cloud storage services. It is also S3 api compatible with
amazon.

Organizations use Swift to store lots of data efficiently, safely, and
cheaply where applications use an special api to interface between the
applications and objects stored in Swift.

Although you can install Swift on a single server, a multiple-server
installation is required for production environments. If you want to
install OpenStack Object Storage (Swift) on a single node for
development or testing purposes, use the Swift All In One instructions
on Ubuntu.

For more information see:
http://swift.openstack.org/development_saio.html .

Support and Troubleshooting
---------------------------

Community Support

-  `OpenStack Mailing list <https://launchpad.net/~openstack>`__

-  `The OpenStack Wiki search <http://wiki.openstack.org>`__

-  `Launchpad bugs area <https://bugs.launchpad.net/nova>`__

-  Join the IRC channel #openstack on freenode.

Resources
---------

-  `Cloud Computing - Service
   models <http://en.wikipedia.org/wiki/Cloud_computing#Service_Models>`__

-  `OpenStack
   Compute <http://www.openstack.org/software/openstack-compute/>`__

-  `OpenStack Image
   Service <http://docs.openstack.org/diablo/openstack-compute/starter/content/GlanceMS-d2s21.html>`__

-  `OpenStack Object Storage Administration
   Guide <http://docs.openstack.org/trunk/openstack-object-storage/admin/content/index.html>`__

-  `Installing OpenStack Object Storage on
   Ubuntu <http://docs.openstack.org/trunk/openstack-object-storage/admin/content/installing-openstack-object-storage-on-ubuntu.html>`__

-  http://cloudglossary.com/

Glossary
--------

The Ubuntu Cloud documentation uses terminology that might be unfamiliar
to some readers. This page is intended to provide a glossary of such
terms and acronyms.

-  *Cloud* - A federated set of physical machines that offer computing
   resources through virtual machines, provisioned and recollected
   dynamically.

-  *IaaS* - Infrastructure as a Service -- Cloud infrastructure
   services, whereby a virtualized environment is delivered as a service
   over the Internet by the provider. The infrastructure can include
   servers, network equipment, and software.

-  *EBS* - Elastic Block Storage.

-  *EC2* - Elastic Compute Cloud. Amazon's pay-by-the-hour,
   pay-by-the-gigabyte public cloud computing offering.

-  *Node* - A node is a physical machine that's capable of running
   virtual machines, running a node controller. Within Ubuntu, this
   generally means that the CPU has VT extensions, and can run the KVM
   hypervisor.

-  *S3* - Simple Storage Service. Amazon's pay-by-the-gigabyte
   persistent storage solution for EC2.

-  *Ubuntu Cloud* - Ubuntu Cloud. Ubuntu's cloud computing solution,
   based on OpenStack.

-  *VM* - Virtual Machine.

-  *VT* - Virtualization Technology. An optional feature of some modern
   CPUs, allowing for accelerated virtual machine hosting.

LXC
===

Containers are a lightweight virtualization technology. They are more
akin to an enhanced chroot than to full virtualization like Qemu or
VMware, both because they do not emulate hardware and because containers
share the same operating system as the host. Therefore containers are
better compared to Solaris zones or BSD jails. Linux-vserver and OpenVZ
are two pre-existing, independently developed implementations of
containers-like functionality for Linux. In fact, containers came about
as a result of the work to upstream the vserver and OpenVZ
functionality. Some vserver and OpenVZ functionality is still missing in
containers, however containers can *boot* many Linux distributions and
have the advantage that they can be used with an un-modified upstream
kernel.

There are two user-space implementations of containers, each exploiting
the same kernel features. Libvirt allows the use of containers through
the LXC driver by connecting to 'lxc:///'. This can be very convenient
as it supports the same usage as its other drivers. The other
implementation, called simply 'LXC', is not compatible with libvirt, but
is more flexible with more userspace tools. It is possible to switch
between the two, though there are peculiarities which can cause
confusion.

In this document we will mainly describe the lxc package. Toward the
end, we will describe how to use the libvirt LXC driver.

In this document, a container name will be shown as CN, C1, or C2.

Installation
------------

The lxc package can be installed using

::

This will pull in the required and recommended dependencies, including
cgroup-lite, lvm2, and debootstrap. To use libvirt-lxc, install
libvirt-bin. LXC and libvirt-lxc can be installed and used at the same
time.

Host Setup
----------

Basic layout of LXC files
~~~~~~~~~~~~~~~~~~~~~~~~~

Following is a description of the files and directories which are
installed and used by LXC.

-  There are two upstart jobs:

   -  ``/etc/init/lxc-net.conf:`` is an optional job which only runs if
      ``
                      /etc/default/lxc`` specifies USE\_LXC\_BRIDGE
      (true by default). It sets up a NATed bridge for containers to
      use.

   -  ``/etc/init/lxc.conf:`` runs if LXC\_AUTO (true by default) is set
      to true in ``/etc/default/lxc``. It looks for entries under
      ``/etc/lxc/auto/`` which are symbolic links to configuration files
      for the containers which should be started at boot.

-  ``/etc/lxc/lxc.conf:`` There is a default container creation
   configuration file, ``/etc/lxc/lxc.conf``, which directs containers
   to use the LXC bridge created by the lxc-net upstart job. If no
   configuration file is specified when creating a container, then this
   one will be used.

-  Examples of other container creation configuration files are found
   under ``/usr/share/doc/lxc/examples``. These show how to create
   containers without a private network, or using macvlan, vlan, or
   other network layouts.

-  The various container administration tools are found under
   ``/usr/bin``.

-  ``/usr/lib/lxc/lxc-init`` is a very minimal and lightweight init
   binary which is used by lxc-execute. Rather than \`booting' a full
   container, it manually mounts a few filesystems, especially
   ``/proc``, and executes its arguments. You are not likely to need to
   manually refer to this file.

-  ``/usr/lib/lxc/templates/`` contains the \`templates' which can be
   used to create new containers of various distributions and flavors.
   Not all templates are currently supported.

-  ``/etc/apparmor.d/lxc/lxc-default`` contains the default Apparmor MAC
   policy which works to protect the host from containers. Please see
   the ? for more information.

-  ``/etc/apparmor.d/usr.bin.lxc-start`` contains a profile to protect
   the host from ``lxc-start`` while it is setting up the container.

-  ``/etc/apparmor.d/lxc-containers`` causes all the profiles defined
   under ``/etc/apparmor.d/lxc`` to be loaded at boot.

-  There are various man pages for the LXC administration tools as well
   as the ``lxc.conf`` container configuration file.

-  ``/var/lib/lxc`` is where containers and their configuration
   information are stored.

-  ``/var/cache/lxc`` is where caches of distribution data are stored to
   speed up multiple container creations.

lxcbr0
~~~~~~

When USE\_LXC\_BRIDGE is set to true in /etc/default/lxc (as it is by
default), a bridge called lxcbr0 is created at startup. This bridge is
given the private address 10.0.3.1, and containers using this bridge
will have a 10.0.3.0/24 address. A dnsmasq instance is run listening on
that bridge, so if another dnsmasq has bound all interfaces before the
lxc-net upstart job runs, lxc-net will fail to start and lxcbr0 will not
exist.

If you have another bridge - libvirt's default virbr0, or a br0 bridge
for your default NIC - you can use that bridge in place of lxcbr0 for
your containers.

Using a separate filesystem for the container store
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

LXC stores container information and (with the default backing store)
root filesystems under ``/var/lib/lxc``. Container creation templates
also tend to store cached distribution information under
``/var/cache/lxc``.

If you wish to use another filesystem than ``/var``, you can mount a
filesystem which has more space into those locations. If you have a disk
dedicated for this, you can simply mount it at ``/var/lib/lxc``. If
you'd like to use another location, like ``/srv``, you can bind mount it
or use a symbolic link. For instance, if ``/srv`` is a large mounted
filesystem, create and symlink two directories:

::

or, using bind mounts:

::

Containers backed by lvm
~~~~~~~~~~~~~~~~~~~~~~~~

It is possible to use LVM partitions as the backing stores for
containers. Advantages of this include flexibility in storage management
and fast container cloning. The tools default to using a VG (volume
group) named *lxc*, but another VG can be used through command line
options. When a LV is used as a container backing store, the container's
configuration file is still ``/var/lib/lxc/CN/config``, but the root fs
entry in that file (*lxc.rootfs*) will point to the lV block device
name, i.e. ``/dev/lxc/CN``.

Containers with directory tree and LVM backing stores can co-exist.

Btrfs
~~~~~

If your host has a btrfs ``/var``, the LXC administration tools will
detect this and automatically exploit it by cloning containers using
btrfs snapshots.

Apparmor
~~~~~~~~

LXC ships with an Apparmor profile intended to protect the host from
accidental misuses of privilege inside the container. For instance, the
container will not be able to write to ``/proc/sysrq-trigger`` or to
most ``/sys`` files.

The ``usr.bin.lxc-start`` profile is entered by running ``lxc-start``.
This profile mainly prevents ``lxc-start`` from mounting new filesystems
outside of the container's root filesystem. Before executing the
container's ``init``, ``LXC`` requests a switch to the container's
profile. By default, this profile is the ``lxc-container-default``
policy which is defined in ``/etc/apparmor.d/lxc/lxc-default``. This
profile prevents the container from accessing many dangerous paths, and
from mounting most filesystems.

If you find that ``lxc-start`` is failing due to a legitimate access
which is being denied by its Apparmor policy, you can disable the
lxc-start profile by doing:

::

    sudo apparmor_parser -R /etc/apparmor.d/usr.bin.lxc-start
    sudo ln -s /etc/apparmor.d/usr.bin.lxc-start /etc/apparmor.d/disabled/

This will make ``lxc-start`` run unconfined, but continue to confine the
container itself. If you also wish to disable confinement of the
container, then in addition to disabling the ``usr.bin.lxc-start``
profile, you must add:

::

    lxc.aa_profile = unconfined

to the container's configuration file. If you wish to run a container in
a custom profile, you can create a new profile under
``/etc/apparmor.d/lxc/``. Its name must start with ``lxc-`` in order for
``lxc-start`` to be allowed to transition to that profile. The
``lxc-default`` profile includes the re-usable abstractions file
``/etc/apparmor.d/abstractions/lxc/container-base``. An easy way to
start a new profile therefore is to do the same, then add extra
permissions at the bottom of your policy.

After creating the policy, load it using:

::

    sudo apparmor_parser -r /etc/apparmor.d/lxc-containers

The profile will automatically be loaded after a reboot, because it is
sourced by the file ``/etc/apparmor.d/lxc-containers``. Finally, to make
container ``CN`` use this new ``lxc-CN-profile``, add the following line
to its configuration file:

::

    lxc.aa_profile = lxc-CN-profile

``lxc-execute`` does not enter an Apparmor profile, but the container it
spawns will be confined.

Control Groups
~~~~~~~~~~~~~~

Control groups (cgroups) are a kernel feature providing hierarchical
task grouping and per-cgroup resource accounting and limits. They are
used in containers to limit block and character device access and to
freeze (suspend) containers. They can be further used to limit memory
use and block i/o, guarantee minimum cpu shares, and to lock containers
to specific cpus. By default, LXC depends on the cgroup-lite package to
be installed, which provides the proper cgroup initialization at boot.
The cgroup-lite package mounts each cgroup subsystem separately under
``/sys/fs/cgroup/SS``, where SS is the subsystem name. For instance the
freezer subsystem is mounted under ``/sys/fs/cgroup/freezer``. LXC
cgroup are kept under ``/sys/fs/cgroup/SS/INIT/lxc``, where INIT is the
init task's cgroup. This is ``/`` by default, so in the end the freezer
cgroup for container CN would be ``/sys/fs/cgroup/freezer/lxc/CN``.

Privilege
~~~~~~~~~

The container administration tools must be run with root user privilege.
A utility called ``lxc-setup`` was written with the intention of
providing the tools with the needed file capabilities to allow non-root
users to run the tools with sufficient privilege. However, as root in a
container cannot yet be reliably contained, this is not worthwhile. It
is therefore recommended to not use ``lxc-setup``, and to provide the
LXC administrators the needed sudo privilege.

The user namespace, which is expected to be available in the next Long
Term Support (LTS) release, will allow containment of the container root
user, as well as reduce the amount of privilege required for creating
and administering containers.

LXC Upstart Jobs
~~~~~~~~~~~~~~~~

As listed above, the lxc package includes two upstart jobs. The first,
``lxc-net``, is always started when the other, ``lxc``, is about to
begin, and stops when it stops. If the USE\_LXC\_BRIDGE variable is set
to false in ``/etc/defaults/lxc``, then it will immediately exit. If it
is true, and an error occurs bringing up the LXC bridge, then the
``lxc`` job will not start. ``lxc-net`` will bring down the LXC bridge
when stopped, unless a container is running which is using that bridge.

The ``lxc`` job starts on runlevel 2-5. If the LXC\_AUTO variable is set
to true, then it will look under ``/etc/lxc`` for containers which
should be started automatically. When the ``lxc`` job is stopped, either
manually or by entering runlevel 0, 1, or 6, it will stop those
containers.

To register a container to start automatically, create a symbolic link
``/etc/lxc/auto/name.conf`` pointing to the container's config file. For
instance, the configuration file for a container ``CN`` is
``/var/lib/lxc/CN/config``. To make that container auto-start, use the
command:

::

Container Administration
------------------------

Creating Containers
~~~~~~~~~~~~~~~~~~~

The easiest way to create containers is using ``lxc-create``. This
script uses distribution-specific templates under
``/usr/lib/lxc/templates/`` to set up container-friendly chroots under
``/var/lib/lxc/CN/rootfs``, and initialize the configuration in
``/var/lib/lxc/CN/fstab`` and ``/var/lib/lxc/CN/config``, where CN is
the container name

The simplest container creation command would look like:

::

This tells lxc-create to use the ubuntu template (-t ubuntu) and to call
the container CN (-n CN). Since no configuration file was specified
(which would have been done with \`-f file'), it will use the default
configuration file under ``/etc/lxc/lxc.conf``. This gives the container
a single veth network interface attached to the lxcbr0 bridge.

The container creation templates can also accept arguments. These can be
listed after --. For instance

::

passes the arguments '-r oneiric1' to the ubuntu template.

Help
^^^^

Help on the lxc-create command can be seen by using\ ``
          lxc-create -h``. However, the templates also take their own
options. If you do

::

then the general ``lxc-create`` help will be followed by help output
specific to the ubuntu template. If no template is specified, then only
help for ``lxc-create`` itself will be shown.

Ubuntu template
^^^^^^^^^^^^^^^

The ubuntu template can be used to create Ubuntu system containers with
any release at least as new as 10.04 LTS. It uses debootstrap to create
a cached container filesystem which gets copied into place each time a
container is created. The cached image is saved and only re-generated
when you create a container using the *-F* (flush) option to the
template, i.e.:

::

The Ubuntu release installed by the template will be the same as that on
the host, unless otherwise specified with the *-r* option, i.e.

::

If you want to create a 32-bit container on a 64-bit host, pass *-a
i386* to the container. If you have the qemu-user-static package
installed, then you can create a container using any architecture
supported by qemu-user-static.

The container will have a user named *ubuntu* whose password is *ubuntu*
and who is a member of the *sudo* group. If you wish to inject a public
ssh key for the *ubuntu* user, you can do so with *-S sshkey.pub*.

You can also *bind* user jdoe from the host into the container using the
*-b jdoe* option. This will copy jdoe's password and shadow entries into
the container, make sure his default group and shell are available, add
him to the sudo group, and bind-mount his home directory into the
container when the container is started.

When a container is created, the ``release-updates`` archive is added to
the container's ``sources.list``, and its package archive will be
updated. If the container release is older than 12.04 LTS, then the
lxcguest package will be automatically installed. Alternatively, if the
*--trim* option is specified, then the lxcguest package will not be
installed, and many services will be removed from the container. This
will result in a faster-booting, but less upgrade-able container.

Ubuntu-cloud template
^^^^^^^^^^^^^^^^^^^^^

The ubuntu-cloud template creates Ubuntu containers by downloading and
extracting the published Ubuntu cloud images. It accepts some of the
same options as the ubuntu template, namely *-r release*, *-S
sshkey.pub*, *-a arch*, and *-F* to flush the cached image. It also
accepts a few extra options. The *-C* option will create a *cloud*
container, configured for use with a metadata service. The *-u* option
accepts a cloud-init user-data file to configure the container on start.
If *-L* is passed, then no locales will be installed. The *-T* option
can be used to choose a tarball location to extract in place of the
published cloud image tarball. Finally the *-i* option sets a host id
for cloud-init, which by default is set to a random string.

Other templates
^^^^^^^^^^^^^^^

The ubuntu and ubuntu-cloud templates are well supported. Other
templates are available however. The debian template creates a Debian
based container, using debootstrap much as the ubuntu template does. By
default it installs a *debian squeeze* image. An alternate release can
be chosen by setting the SUITE environment variable, i.e.:

::

To purge the container image cache, call the template directly and pass
it the *--clean* option.

::

A fedora template exists, which creates containers based on fedora
releases <= 14. Fedora release 15 and higher are based on systemd, which
the template is not yet able to convert into a container-bootable setup.
Before the fedora template is able to run, you'll need to make sure that
``yum`` and ``curl`` are installed. A fedora 12 container can be created
with

::

A OpenSuSE template exists, but it requires the ``zypper`` program,
which is not yet packaged. The OpenSuSE template is therefore not
supported.

Two more templates exist mainly for experimental purposes. The busybox
template creates a very small system container based entirely on
busybox. The sshd template creates an application container running sshd
in a private network namespace. The host's library and binary
directories are bind-mounted into the container, though not its
``/home`` or ``/root``. To create, start, and ssh into an ssh container,
you might:

::

Backing Stores
^^^^^^^^^^^^^^

By default, ``lxc-create`` places the container's root filesystem as a
directory tree at ``/var/lib/lxc/CN/rootfs.`` Another option is to use
LVM logical volumes. If a volume group named *lxc* exists, you can
create an lvm-backed container called CN using:

::

If you want to use a volume group named schroots, with a 5G xfs
filesystem, then you would use

::

Cloning
~~~~~~~

For rapid provisioning, you may wish to customize a canonical container
according to your needs and then make multiple copies of it. This can be
done with the ``lxc-clone`` program. Given an existing container called
C1, a new container called C2 can be created using

::

If ``/var/lib/lxc`` is a btrfs filesystem, then ``lxc-clone`` will
create C2's filesystem as a snapshot of C1's. If the container's root
filesystem is lvm backed, then you can specify the *-s* option to create
the new rootfs as a lvm snapshot of the original as follows:

::

Both lvm and btrfs snapshots will provide fast cloning with very small
initial disk usage.

Starting and stopping
~~~~~~~~~~~~~~~~~~~~~

    **Note**

    The default login/password combination for the newly created
    container is ubuntu/ubuntu.

To start a container, use ``lxc-start -n CN``. By default ``lxc-start``
will execute ``/sbin/init`` in the container. You can provide a
different program to execute, plus arguments, as further arguments to
``lxc-start``:

::

If you do not specify the *-d* (daemon) option, then you will see a
console (on the container's ``/dev/console``, see ? for more
information) on the terminal. If you specify the *-d* option, you will
not see that console, and lxc-start will immediately exit success - even
if a later part of container startup has failed. You can use
``lxc-wait`` or ``lxc-monitor`` (see ?) to check on the success or
failure of the container startup.

To obtain LXC debugging information, use *-o filename -l debuglevel*,
for instance:

::

Finally, you can specify configuration parameters inline using *-s*.
However, it is generally recommended to place them in the container's
configuration file instead. Likewise, an entirely alternate config file
can be specified with the *-f* option, but this is not generally
recommended.

While ``lxc-start`` runs the container's ``/sbin/init``, ``lxc-execute``
uses a minimal init program called ``lxc-init``, which attempts to mount
``/proc``, ``/dev/mqueue``, and ``/dev/shm``, executes the programs
specified on the command line, and waits for those to finish executing.
``lxc-start`` is intended to be used for *system containers*, while
``lxc-execute`` is intended for *application containers* (see `this
article <https://www.ibm.com/developerworks/linux/library/l-lxc-containers/>`__
for more).

You can stop a container several ways. You can use ``shutdown``,
``poweroff`` and ``reboot`` while logged into the container. To cleanly
shut down a container externally (i.e. from the host), you can issue the
``sudo lxc-shutdown -n CN`` command. This takes an optional timeout
value. If not specified, the command issues a SIGPWR signal to the
container and immediately returns. If the option is used, as in
``sudo lxc-shutdown -n CN -t 10``, then the command will wait the
specified number of seconds for the container to cleanly shut down.
Then, if the container is still running, it will kill it (and any
running applications). You can also immediately kill the container
(without any chance for applications to cleanly shut down) using
``sudo lxc-stop -n CN``. Finally, ``lxc-kill`` can be used more
generally to send any signal number to the container's init.

While the container is shutting down, you can expect to see some
(harmless) error messages, as follows:

::

    $ sudo poweroff
    [sudo] password for ubuntu: =

    $ =

    Broadcast message from ubuntu@cn1
            (/dev/lxc/console) at 18:17 ...

    The system is going down for power off NOW!
     * Asking all remaining processes to terminate...
       ...done.
     * All processes ended within 1 seconds....
       ...done.
     * Deconfiguring network interfaces...
       ...done.
     * Deactivating swap...
       ...fail!
    umount: /run/lock: not mounted
    umount: /dev/shm: not mounted
    mount: / is busy
     * Will now halt

A container can be frozen with ``sudo lxc-freeze -n
        CN``. This will block all its processes until the container is
later unfrozen using ``sudo lxc-unfreeze -n
        CN``.

Lifecycle management hooks
~~~~~~~~~~~~~~~~~~~~~~~~~~

Beginning with Ubuntu 12.10, it is possible to define hooks to be
executed at specific points in a container's lifetime:

-  Pre-start hooks are run in the host's namespace before the container
   ttys, consoles, or mounts are up. If any mounts are done in this
   hook, they should be cleaned up in the post-stop hook.

-  Pre-mount hooks are run in the container's namespaces, but before the
   root filesystem has been mounted. Mounts done in this hook will be
   automatically cleaned up when the container shuts down.

-  Mount hooks are run after the container filesystems have been
   mounted, but before the container has called ``pivot_root`` to change
   its root filesystem.

-  Start hooks are run immediately before executing the container's
   init. Since these are executed after pivoting into the container's
   filesystem, the command to be executed must be copied into the
   container's filesystem.

-  Post-stop hooks are executed after the container has been shut down.

If any hook returns an error, the container's run will be aborted. Any
*post-stop* hook will still be executed. Any output generated by the
script will be logged at the debug priority.

See ? for the configuration file format with which to specify hooks.
Some sample hooks are shipped with the lxc package to serve as an
example of how to write and use such hooks.

Monitoring container status
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Two commands are available to monitor container state changes.
``lxc-monitor`` monitors one or more containers for any state changes.
It takes a container name as usual with the *-n* option, but in this
case the container name can be a posix regular expression to allow
monitoring desirable sets of containers. ``lxc-monitor`` continues
running as it prints container changes. ``lxc-wait`` waits for a
specific state change and then exits. For instance,

::

would print all state changes to any containers matching the listed
regular expression, whereas

::

will wait until container cont1 enters state STOPPED or state FROZEN and
then exit.

Consoles
~~~~~~~~

Containers have a configurable number of consoles. One always exists on
the container's ``/dev/console``. This is shown on the terminal from
which you ran ``lxc-start``, unless the *-d* option is specified. The
output on ``/dev/console`` can be redirected to a file using the *-c
console-file* option to ``lxc-start``. The number of extra consoles is
specified by the ``lxc.tty`` variable, and is usually set to 4. Those
consoles are shown on ``/dev/ttyN`` (for 1 <= N <= 4). To log into
console 3 from the host, use

::

or if the *-t N* option is not specified, an unused console will be
automatically chosen. To exit the console, use the escape sequence
Ctrl-a q. Note that the escape sequence does not work in the console
resulting from ``lxc-start`` without the *-d* option.

Each container console is actually a Unix98 pty in the host's (not the
guest's) pty mount, bind-mounted over the guest's ``/dev/ttyN`` and
``/dev/console``. Therefore, if the guest unmounts those or otherwise
tries to access the actual character device ``4:N``, it will not be
serving getty to the LXC consoles. (With the default settings, the
container will not be able to access that character device and getty
will therefore fail.) This can easily happen when a boot script blindly
mounts a new ``/dev``.

Container Inspection
~~~~~~~~~~~~~~~~~~~~

Several commands are available to gather information on existing
containers. ``lxc-ls`` will report all existing containers in its first
line of output, and all running containers in the second line.
``lxc-list`` provides the same information in a more verbose format,
listing running containers first and stopped containers next. ``lxc-ps``
will provide lists of processes in containers. To provide ``ps``
arguments to ``lxc-ps``, prepend them with ``--``. For instance, for
listing of all processes in container plain,

::

``lxc-info`` provides the state of a container and the pid of its init
process. ``lxc-cgroup`` can be used to query or set the values of a
container's control group limits and information. This can be more
convenient than interacting with the ``cgroup`` filesystem. For
instance, to query the list of devices which a running container is
allowed to access, you could use

::

or to add mknod, read, and write access to ``/dev/sda``,

::

and, to limit it to 300M of RAM,

::

``lxc-netstat`` executes ``netstat`` in the running container, giving
you a glimpse of its network state.

``lxc-backup`` will create backups of the root filesystems of all
existing containers (except lvm-based ones), using ``rsync`` to back the
contents up under ``/var/lib/lxc/CN/rootfs.backup.1``. These backups can
be restored using ``lxc-restore.`` However, ``lxc-backup`` and
``lxc-restore`` are fragile with respect to customizations and therefore
their use is not recommended.

Destroying containers
~~~~~~~~~~~~~~~~~~~~~

Use ``lxc-destroy`` to destroy an existing container.

::

If the container is running, ``lxc-destroy`` will exit with a message
informing you that you can force stopping and destroying the container
with

::

Advanced namespace usage
~~~~~~~~~~~~~~~~~~~~~~~~

One of the Linux kernel features used by LXC to create containers is
private namespaces. Namespaces allow a set of tasks to have private
mappings of names to resources for things like pathnames and process
IDs. (See ? for a link to more information). Unlike control groups and
other mount features which are also used to create containers,
namespaces cannot be manipulated using a filesystem interface.
Therefore, LXC ships with the ``lxc-unshare`` program, which is mainly
for testing. It provides the ability to create new tasks in private
namespaces. For instance,

::

creates a bash shell with private pid and mount namespaces. In this
shell, you can do

::

    root@ubuntu:~# mount -t proc proc /proc
    root@ubuntu:~# ps -ef
    UID        PID  PPID  C STIME TTY          TIME CMD
    root         1     0  6 10:20 pts/9    00:00:00 /bin/bash
    root       110     1  0 10:20 pts/9    00:00:00 ps -ef

so that ``ps`` shows only the tasks in your new namespace.

Ephemeral containers
~~~~~~~~~~~~~~~~~~~~

Ephemeral containers are one-time containers. Given an existing
container CN, you can run a command in an ephemeral container created
based on CN, with the host's jdoe user bound into the container, using:

::

When the job is finished, the container will be discarded.

Container Commands
~~~~~~~~~~~~~~~~~~

Following is a table of all container commands:

+----------------------+-----------------------------------------------------+
| Command              | Synopsis                                            |
+======================+=====================================================+
| lxc-attach           | (NOT SUPPORTED) Run a command in a running          |
|                      | container                                           |
+----------------------+-----------------------------------------------------+
| lxc-backup           | Back up the root filesystems for all lvm-backed     |
|                      | containers                                          |
+----------------------+-----------------------------------------------------+
| lxc-cgroup           | View and set container control group settings       |
+----------------------+-----------------------------------------------------+
| lxc-checkconfig      | Verify host support for containers                  |
+----------------------+-----------------------------------------------------+
| lxc-checkpoint       | (NOT SUPPORTED) Checkpoint a running container      |
+----------------------+-----------------------------------------------------+
| lxc-clone            | Clone a new container from an existing one          |
+----------------------+-----------------------------------------------------+
| lxc-console          | Open a console in a running container               |
+----------------------+-----------------------------------------------------+
| lxc-create           | Create a new container                              |
+----------------------+-----------------------------------------------------+
| lxc-destroy          | Destroy an existing container                       |
+----------------------+-----------------------------------------------------+
| lxc-execute          | Run a command in a (not running) application        |
|                      | container                                           |
+----------------------+-----------------------------------------------------+
| lxc-freeze           | Freeze a running container                          |
+----------------------+-----------------------------------------------------+
| lxc-info             | Print information on the state of a container       |
+----------------------+-----------------------------------------------------+
| lxc-kill             | Send a signal to a container's init                 |
+----------------------+-----------------------------------------------------+
| lxc-list             | List all containers                                 |
+----------------------+-----------------------------------------------------+
| lxc-ls               | List all containers with shorter output than        |
|                      | lxc-list                                            |
+----------------------+-----------------------------------------------------+
| lxc-monitor          | Monitor state changes of one or more containers     |
+----------------------+-----------------------------------------------------+
| lxc-netstat          | Execute netstat in a running container              |
+----------------------+-----------------------------------------------------+
| lxc-ps               | View process info in a running container            |
+----------------------+-----------------------------------------------------+
| lxc-restart          | (NOT SUPPORTED) Restart a checkpointed container    |
+----------------------+-----------------------------------------------------+
| lxc-restore          | Restore containers from backups made by lxc-backup  |
+----------------------+-----------------------------------------------------+
| lxc-setcap           | (NOT RECOMMENDED) Set file capabilities on LXC      |
|                      | tools                                               |
+----------------------+-----------------------------------------------------+
| lxc-setuid           | (NOT RECOMMENDED) Set or remove setuid bits on LXC  |
|                      | tools                                               |
+----------------------+-----------------------------------------------------+
| lxc-shutdown         | Safely shut down a container                        |
+----------------------+-----------------------------------------------------+
| lxc-start            | Start a stopped container                           |
+----------------------+-----------------------------------------------------+
| lxc-start-ephemeral  | Start an ephemeral (one-time) container             |
+----------------------+-----------------------------------------------------+
| lxc-stop             | Immediately stop a running container                |
+----------------------+-----------------------------------------------------+
| lxc-unfreeze         | Unfreeze a frozen container                         |
+----------------------+-----------------------------------------------------+
| lxc-unshare          | Testing tool to manually unshare namespaces         |
+----------------------+-----------------------------------------------------+
| lxc-version          | Print the version of the LXC tools                  |
+----------------------+-----------------------------------------------------+
| lxc-wait             | Wait for a container to reach a particular state    |
+----------------------+-----------------------------------------------------+

Table: Container commands

Configuration File
------------------

LXC containers are very flexible. The Ubuntu lxc package sets defaults
to make creation of Ubuntu system containers as simple as possible. If
you need more flexibility, this chapter will show how to fine-tune your
containers as you need.

Detailed information is available in the ``lxc.conf(5)`` man page. Note
that the default configurations created by the ubuntu templates are
reasonable for a system container and usually do not need customization.

Choosing configuration files and options
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The container setup is controlled by the LXC configuration options.
Options can be specified at several points:

-  During container creation, a configuration file can be specified.
   However, creation templates often insert their own configuration
   options, so we usually specify only network configuration options at
   this point. For other configuration, it is usually better to edit the
   configuration file after container creation.

-  The file ``/var/lib/lxc/CN/config`` is used at container startup by
   default.

-  ``lxc-start`` accepts an alternate configuration file with the *-f
   filename* option.

-  Specific configuration variables can be overridden at ``lxc-start``
   using *-s key=value*. It is generally better to edit the container
   configuration file.

Network Configuration
~~~~~~~~~~~~~~~~~~~~~

Container networking in LXC is very flexible. It is triggered by the
``lxc.network.type`` configuration file entries. If no such entries
exist, then the container will share the host's networking stack.
Services and connections started in the container will be using the
host's IP address. If at least one ``lxc.network.type`` entry is
present, then the container will have a private (layer 2) network stack.
It will have its own network interfaces and firewall rules. There are
several options for ``lxc.network.type``:

-  ``lxc.network.type=empty``: The container will have no network
   interfaces other than loopback.

-  ``lxc.network.type=veth``: This is the default when using the ubuntu
   or ubuntu-cloud templates, and creates a veth network tunnel. One end
   of this tunnel becomes the network interface inside the container.
   The other end is attached to a bridged on the host. Any number of
   such tunnels can be created by adding more ``lxc.network.type=veth``
   entries in the container configuration file. The bridge to which the
   host end of the tunnel will be attached is specified with
   ``lxc.network.link = lxcbr0``.

-  ``lxc.network.type=phys`` A physical network interface (i.e. eth2) is
   passed into the container.

Two other options are to use vlan or macvlan, however their use is more
complicated and is not described here. A few other networking options
exist:

-  ``lxc.network.flags`` can only be set to *up* and ensures that the
   network interface is up.

-  ``lxc.network.hwaddr`` specifies a mac address to assign to the nic
   inside the container.

-  ``lxc.network.ipv4`` and ``lxc.network.ipv6`` set the respective IP
   addresses, if those should be static.

-  ``lxc.network.name`` specifies a name to assign inside the container.
   If this is not specified, a good default (i.e. eth0 for the first
   nic) is chosen.

-  ``lxc.network.lxcscript.up`` specifies a script to be called after
   the host side of the networking has been set up. See the
   ``lxc.conf(5)`` manual page for details.

Control group configuration
~~~~~~~~~~~~~~~~~~~~~~~~~~~

Cgroup options can be specified using ``lxc.cgroup`` entries.
``lxc.cgroup.subsystem.item = value`` instructs LXC to set cgroup
``subsystem``'s ``item`` to ``value``. It is perhaps simpler to realize
that this will simply write ``value`` to the file ``item`` for the
container's control group for subsystem ``subsystem``. For instance, to
set the memory limit to 320M, you could add

::

which will cause 320000000 to be written to the file
``/sys/fs/cgroup/memory/lxc/CN/limit_in_bytes``.

Rootfs, mounts and fstab
~~~~~~~~~~~~~~~~~~~~~~~~

An important part of container setup is the mounting of various
filesystems into place. The following is an example configuration file
excerpt demonstrating the commonly used configuration options:

::

The first line says that the container's root filesystem is already
mounted at ``/var/lib/lxc/CN/rootfs``. If the filesystem is a block
device (such as an LVM logical volume), then the path to the block
device must be given instead.

Each ``lxc.mount.entry`` line should contain an item to mount in valid
fstab format. The target directory should be prefixed by
``/var/lib/lxc/CN/rootfs``, even if ``lxc.rootfs`` points to a block
device.

Finally, ``lxc.mount`` points to a file, in fstab format, containing
further items to mount. Note that all of these entries will be mounted
by the host before the container init is started. In this way it is
possible to bind mount various directories from the host into the
container.

Other configuration options
~~~~~~~~~~~~~~~~~~~~~~~~~~~

-  ``lxc.cap.drop`` can be used to prevent the container from having or
   ever obtaining the listed capabilities. For instance, including

   ::

   will prevent the container from mounting filesystems, as well as all
   other actions which require cap\_sys\_admin. See the
   ``capabilities(7)`` manual page for a list of capabilities and their
   meanings.

-  ``lxc.aa_profile = lxc-CN-profile`` specifies a custom Apparmor
   profile in which to start the container. See ? for more information.

-  ``lxc.console=/path/to/consolefile`` will cause console messages to
   be written to the specified file.

-  ``lxc.arch`` specifies the architecture for the container, for
   instance x86, or x86\_64.

-  ``lxc.tty=5`` specifies that 5 consoles (in addition to
   ``/dev/console``) should be created. That is, consoles will be
   available on ``/dev/tty1`` through ``/dev/tty5``. The ubuntu
   templates set this value to 4.

-  ``lxc.pts=1024`` specifies that the container should have a private
   (Unix98) devpts filesystem mount. If this is not specified, then the
   container will share ``/dev/pts`` with the host, which is rarely
   desired. The number 1024 means that 1024 ptys should be allowed in
   the container, however this number is currently ignored. Before
   starting the container init, LXC will do (essentially) a

   ::

   inside the container. It is important to realize that the container
   should not mount devpts filesystems of its own. It may safely do bind
   or move mounts of its mounted ``/dev/pts``. But if it does

   ::

   it will remount the host's devpts instance. If it adds the
   newinstance mount option, then it will mount a new private (empty)
   instance. In neither case will it remount the instance which was set
   up by LXC. For this reason, and to prevent the container from using
   the host's ptys, the default Apparmor policy will not allow
   containers to mount devpts filesystems after the container's init has
   been started.

-  ``lxc.devttydir`` specifies a directory under ``/dev`` in which LXC
   will create its console devices. If this option is not specified,
   then the ptys will be bind-mounted over ``/dev/console`` and
   ``/dev/ttyN.`` However, rare package updates may try to blindly *rm
   -f* and then *mknod* those devices. They will fail (because the file
   has been bind-mounted), causing the package update to fail. When
   ``lxc.devttydir`` is set to LXC, for instance, then LXC will
   bind-mount the console ptys onto ``/dev/lxc/console`` and
   ``/dev/lxc/ttyN,`` and subsequently symbolically link them to
   ``/dev/console`` and ``/dev/ttyN.`` This allows the package updates
   to succeed, at the risk of making future gettys on those consoles
   fail until the next reboot. This problem will be ideally solved with
   device namespaces.

-  The ``lxc.hook.`` options specify programs to run at various points
   in a container's life cycle. See ? for more information on these
   hooks. To have multiple hooks called at any point, list them in
   multiple entries. The possible values, whose precise meanings are
   described in ?, are

   -  ``lxc.hook.pre-start``

   -  ``lxc.hook.pre-mount``

   -  ``lxc.hook.mount``

   -  ``lxc.hook.start``

   -  ``lxc.hook.post-stop``

-  The ``lxc.include`` option specifies another configuration file to be
   loaded. This allows common configuration sections to be defined once
   and included by several containers, simplifying updates of the common
   section.

-  The ``lxc.seccomp`` option (introduced with Ubuntu 12.10) specifies a
   file containing a *seccomp* policy to load. See ? for more
   information on seccomp in lxc.

Updates in Ubuntu containers
----------------------------

Because of some limitations which are placed on containers, package
upgrades at times can fail. For instance, a package install or upgrade
might fail if it is not allowed to create or open a block device. This
often blocks all future upgrades until the issue is resolved. In some
cases, you can work around this by chrooting into the container, to
avoid the container restrictions, and completing the upgrade in the
chroot.

Some of the specific things known to occasionally impede package
upgrades include:

-  The container modifications performed when creating containers with
   the --trim option.

-  Actions performed by lxcguest. For instance, because
   ``/lib/init/fstab`` is bind-mounted from another file, mountall
   upgrades which insist on replacing that file can fail.

-  The over-mounting of console devices with ptys from the host can
   cause trouble with udev upgrades.

-  Apparmor policy and devices cgroup restrictions can prevent package
   upgrades from performing certain actions.

-  Capabilities dropped by use of ``lxc.cap.drop`` can likewise stop
   package upgrades from performing certain actions.

Libvirt LXC
-----------

Libvirt is a powerful hypervisor management solution with which you can
administer Qemu, Xen and LXC virtual machines, both locally and remote.
The libvirt LXC driver is a separate implementation from what we
normally call *LXC*. A few differences include:

-  Configuration is stored in xml format

-  There no tools to facilitate container creation

-  By default there is no console on ``/dev/console``

-  There is no support (yet) for container reboot or full shutdown

Converting a LXC container to libvirt-lxc
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

? showed how to create LXC containers. If you've created a valid LXC
container in this way, you can manage it with libvirt. Fetch a sample
xml file from

::

Edit this file to replace the container name and root filesystem
locations. Then you can define the container with:

::

Creating a container from cloud image
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

If you prefer to create a pristine new container just for LXC, you can
download an ubuntu cloud image, extract it, and point a libvirt LXC xml
file to it. For instance, find the url for a root tarball for the latest
daily Ubuntu 12.04 LTS cloud image using

::

Extract the downloaded tarball, for instance

::

Download the xml template

::

In the xml template, replace the name o1 with c1 and the source
directory ``/var/lib/lxc/o1/rootfs`` with ``$HOME/c1``. Then define the
container using

::

Interacting with libvirt containers
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

As we've seen, you can create a libvirt-lxc container using

::

To start a container called *container*, use

::

To stop a running container, use

::

Note that whereas the ``lxc-destroy`` command deletes the container, the
``virsh destroy`` command stops a running container. To delete the
container definition, use

::

To get a console to a running container, use

::

Exit the console by simultaneously pressing control and ].

The lxcguest package
--------------------

In the 11.04 (Natty) and 11.10 (Oneiric) releases of Ubuntu, a package
was introduced called *lxcguest*. An unmodified root image could not be
safely booted inside a container, but an image with the lxcguest package
installed could be booted as a container, on bare hardware, or in a Xen,
kvm, or VMware virtual machine.

As of the 12.04 LTS release, the work previously done by the lxcguest
package was pushed into the core packages, and the lxcguest package was
removed. As a result, an unmodified 12.04 LTS image can be booted as a
container, on bare hardware, or in a Xen, kvm, or VMware virtual
machine. To use an older release, the lxcguest package should still be
used.

Python api
----------

As of 12.10 (Quantal) a python3-lxc package is available which provides
a python module, called ``lxc``, for managing lxc containers. An example
python session to create and start an Ubuntu container called ``C1``,
then wait until it has been shut down, would look like:

::

    # sudo python3
    Python 3.2.3 (default, Aug 28 2012, 08:26:03)
    [GCC 4.7.1 20120814 (prerelease)] on linux2
    Type "help", "copyright", "credits" or "license" for more information.
    >>> import lxc
    __main__:1: Warning: The python-lxc API isn't yet stable and may change at any p
    oint in the future.
    >>> c=lxc.Container("C1")
    >>> c.create("ubuntu")
    True
    >>> c.start()
    True
    >>> c.wait("STOPPED")
    True

Debug information for containers started with the python API will be
placed in ``/var/log/lxccontainer.log``.

Security
--------

A namespace maps ids to resources. By not providing a container any id
with which to reference a resource, the resource can be protected. This
is the basis of some of the security afforded to container users. For
instance, IPC namespaces are completely isolated. Other namespaces,
however, have various *leaks* which allow privilege to be
inappropriately exerted from a container into another container or to
the host.

By default, LXC containers are started under a Apparmor policy to
restrict some actions. However, while stronger security is a goal for
future releases, in 12.04 LTS the goal of the Apparmor policy is not to
stop malicious actions but rather to stop accidental harm of the host by
the guest. The details of AppArmor integration with lxc are in section ?

Exploitable system calls
~~~~~~~~~~~~~~~~~~~~~~~~

It is a core container feature that containers share a kernel with the
host. Therefore if the kernel contains any exploitable system calls the
container can exploit these as well. Once the container controls the
kernel it can fully control any resource known to the host.

Since Ubuntu 12.10 (Quantal) a container can also be constrained by a
seccomp filter. Seccomp is a new kernel feature which filters the system
calls which may be used by a task and its children. While improved and
simplified policy management is expected in the near future, the current
policy consists of a simple whitelist of system call numbers. The policy
file begins with a version number (which must be 1) on the first line
and a policy type (which must be 'whitelist') on the second line. It is
followed by a list of numbers, one per line.

In general to run a full distribution container a large number of system
calls will be needed. However for application containers it may be
possible to reduce the number of available system calls to only a few.
Even for system containers running a full distribution security gains
may be had, for instance by removing the 32-bit compatibility system
calls in a 64-bit container. See ? for details of how to configure a
container to use seccomp. By default, no seccomp policy is loaded.

Resources
---------

-  The DeveloperWorks article `LXC: Linux container
   tools <https://www.ibm.com/developerworks/linux/library/l-lxc-containers/>`__
   was an early introduction to the use of containers.

-  The `Secure Containers
   Cookbook <http://www.ibm.com/developerworks/linux/library/l-lxc-security/index.html>`__
   demonstrated the use of security modules to make containers more
   secure.

-  Manual pages referenced above can be found at:

   ::


-  The upstream LXC project is hosted at
   `Sourceforge <http://lxc.sf.net>`__.

-  LXC security issues are listed and discussed at `the LXC Security
   wiki page <http://wiki.ubuntu.com/LxcSecurity>`__

-  For more on namespaces in Linux, see: S. Bhattiprolu, E. W.
   Biederman, S. E. Hallyn, and D. Lezcano. Virtual Servers and Check-
   point/Restart in Mainstream Linux. SIGOPS Op- erating Systems Review,
   42(5), 2008.


