Installation
============

This chapter provides a quick overview of installing Ubuntu DISTRO-REV
Server Edition. For more detailed instructions, please refer to the
`Ubuntu Installation
Guide <https://help.ubuntu.com/&distro-rev-short;/installation-guide/>`__.

Preparing to Install
====================

This section explains various aspects to consider before starting the
installation.

System Requirements
-------------------

Ubuntu DISTRO-REV Server Edition supports three (3) major architectures:
Intel x86, AMD64 and ARM. The table below lists recommended hardware
specifications. Depending on your needs, you might manage with less than
this. However, most users risk being frustrated if they ignore these
suggestions.

+---------------+---------------+---------------+---------------+------------------+
| Install Type  | CPU           | RAM           | Hard Drive    |
|               |               |               | Space         |
+===============+===============+===============+===============+==================+
| Server        | 1 gigahertz   | 512 megabytes | 1 gigabyte    | 1.75 gigabytes   |
| (Standard)    |               |               |               |                  |
+---------------+---------------+---------------+---------------+------------------+
| Server        | 300 megahertz | 128 megabytes | 700 megabytes | 1.4 gigabytes    |
| (Minimal)     |               |               |               |                  |
+---------------+---------------+---------------+---------------+------------------+

Table: Recommended Minimum Requirements

    **Note**

    A virtual installation requires a minimum of 256 megabytes of RAM
    during the installation phase.

The Server Edition provides a common base for all sorts of server
applications. It is a minimalist design providing a platform for the
desired services, such as file/print services, web hosting, email
hosting, etc.

Server and Desktop Differences
------------------------------

There are a few differences between the *Ubuntu Server Edition* and the
*Ubuntu Desktop Edition*. It should be noted that both editions use the
same apt repositories, making it just as easy to install a *server*
application on the Desktop Edition as it is on the Server Edition.

The differences between the two editions are the lack of an X window
environment in the Server Edition and the installation process.

Kernel Differences:
~~~~~~~~~~~~~~~~~~~

Ubuntu version 10.10 and prior, actually had different kernels for the
server and desktop editions. Ubuntu no longer has separate -server and
-generic kernel flavors. These have been merged into a single -generic
kernel flavor to help reduce the maintenance burden over the life of the
release.

    **Note**

    When running a 64-bit version of Ubuntu on 64-bit processors you are
    not limited by memory addressing space.

To see all kernel configuration options you can look through
``/boot/config-LINUX-KERNEL-VERSION-server``. Also, `Linux Kernel in a
Nutshell <http://www.kroah.com/lkn/>`__ is a great resource on the
options available.

Backing Up
----------

-  Before installing Ubuntu Server Edition you should make sure all data
   on the system is backed up. See ? for backup options.

   If this is not the first time an operating system has been installed
   on your computer, it is likely you will need to re-partition your
   disk to make room for Ubuntu.

   Any time you partition your disk, you should be prepared to lose
   everything on the disk should you make a mistake or something goes
   wrong during partitioning. The programs used in installation are
   quite reliable, most have seen years of use, but they also perform
   destructive actions.

Installing from CD
==================

The basic steps to install Ubuntu Server Edition from CD are the same as
those for installing any operating system from CD. Unlike the *Desktop
Edition*, the *Server Edition* does not include a graphical installation
program. The Server Edition uses a console menu based process instead.

-  First, download and burn the appropriate ISO file from the `Ubuntu
   web site <http://www.ubuntu.com/download/server/download>`__.

-  Boot the system from the CD-ROM drive.

-  At the boot prompt you will be asked to select a language.

-  From the main boot menu there are some additional options to install
   Ubuntu Server Edition. You can install a basic Ubuntu Server, check
   the CD-ROM for defects, check the system's RAM, boot from first hard
   disk, or rescue a broken system. The rest of this section will cover
   the basic Ubuntu Server install.

-  The installer asks for which language it should use. Afterwards, you
   are asked to select your location.

-  Next, the installation process begins by asking for your keyboard
   layout. You can ask the installer to attempt auto-detecting it, or
   you can select it manually from a list.

-  The installer then discovers your hardware configuration, and
   configures the network settings using DHCP. If you do not wish to use
   DHCP at the next screen choose "Go Back", and you have the option to
   "Configure the network manually".

-  Next, the installer asks for the system's hostname and Time Zone.

-  You can then choose from several options to configure the hard drive
   layout. Afterwards you are asked for which disk to install to. You
   may get confirmation prompts before rewriting the partition table or
   setting up LVM depending on disk layout. If you choose LVM, you will
   be asked for the size of the root logical volume. For advanced disk
   options see ?.

-  The Ubuntu base system is then installed.

-  A new user is set up; this user will have *root* access through the
   sudo utility.

-  After the user settings have been completed, you will be asked to
   encrypt your ``home`` directory.

-  The next step in the installation process is to decide how you want
   to update the system. There are three options:

   -  *No automatic updates*: this requires an administrator to log into
      the machine and manually install updates.

   -  *Install security updates automatically*: this will install the
      unattended-upgrades package, which will install security updates
      without the intervention of an administrator. For more details see
      ?.

   -  *Manage the system with Landscape*: Landscape is a paid service
      provided by Canonical to help manage your Ubuntu machines. See the
      `Landscape <http://www.canonical.com/projects/landscape>`__ site
      for details.

-  You now have the option to install, or not install, several package
   tasks. See ? for details. Also, there is an option to launch aptitude
   to choose specific packages to install. For more information see ?.

-  Finally, the last step before rebooting is to set the clock to UTC.

    **Note**

    If at any point during installation you are not satisfied by the
    default setting, use the "Go Back" function at any prompt to be
    brought to a detailed installation menu that will allow you to
    modify the default settings.

At some point during the installation process you may want to read the
help screen provided by the installation system. To do this, press F1.

Once again, for detailed instructions see the `Ubuntu Installation
Guide <https://help.ubuntu.com/&distro-rev-short;/installation-guide/>`__.

Package Tasks
-------------

During the Server Edition installation you have the option of installing
additional packages from the CD. The packages are grouped by the type of
service they provide.

-  DNS server: Selects the BIND DNS server and its documentation.

-  LAMP server: Selects a ready-made Linux/Apache/MySQL/PHP server.

-  Mail server: This task selects a variety of packages useful for a
   general purpose mail server system.

-  OpenSSH server: Selects packages needed for an OpenSSH server.

-  PostgreSQL database: This task selects client and server packages for
   the PostgreSQL database.

-  Print server: This task sets up your system to be a print server.

-  Samba File server: This task sets up your system to be a Samba file
   server, which is especially suitable in networks with both Windows
   and Linux systems.

-  Tomcat Java server: Installs Apache Tomcat and needed dependencies.

-  Virtual Machine host: Includes packages needed to run KVM virtual
   machines.

-  Manually select packages: Executes aptitude allowing you to
   individually select packages.

Installing the package groups is accomplished using the tasksel utility.
One of the important differences between Ubuntu (or Debian) and other
GNU/Linux distribution is that, when installed, a package is also
configured to reasonable defaults, eventually prompting you for
additional required information. Likewise, when installing a task, the
packages are not only installed, but also configured to provided a fully
integrated service.

Once the installation process has finished you can view a list of
available tasks by entering the following from a terminal prompt:

::

    **Note**

    The output will list tasks from other Ubuntu based distributions
    such as Kubuntu and Edubuntu. Note that you can also invoke the
    ``tasksel`` command by itself, which will bring up a menu of the
    different tasks available.

You can view a list of which packages are installed with each task using
the *--task-packages* option. For example, to list the packages
installed with the *DNS Server* task enter the following:

::

The output of the command should list:

::

    bind9-doc 
    bind9utils 
    bind9

If you did not install one of the tasks during the installation process,
but for example you decide to make your new LAMP server a DNS server as
well, simply insert the installation CD and from a terminal:

::

Upgrading
=========

There are several ways to upgrade from one Ubuntu release to another.
This section gives an overview of the recommended upgrade method.

do-release-upgrade
------------------

The recommended way to upgrade a Server Edition installation is to use
the do-release-upgrade utility. Part of the *update-manager-core*
package, it does not have any graphical dependencies and is installed by
default.

Debian based systems can also be upgraded by using
``apt-get dist-upgrade``. However, using do-release-upgrade is
recommended because it has the ability to handle system configuration
changes sometimes needed between releases.

To upgrade to a newer release, from a terminal prompt enter:

::

It is also possible to use do-release-upgrade to upgrade to a
development version of Ubuntu. To accomplish this use the *-d* switch:

::

    **Warning**

    Upgrading to a development release is *not* recommended for
    production environments.

Advanced Installation
=====================

Software RAID
-------------

Redundant Array of Independent Disks "RAID" is a method of using
multiple disks to provide different balances of increasing data
reliability and/or increasing input/output performance, depending on the
RAID level being used. RAID is implemented in either software (where the
operating system knows about both drives and actively maintains both of
them) or hardware (where a special controller makes the OS think there's
only one drive and maintains the drives 'invisibly').

The RAID software included with current versions of Linux (and Ubuntu)
is based on the 'mdadm' driver and works very well, better even than
many so-called 'hardware' RAID controllers. This section will guide you
through installing Ubuntu Server Edition using two RAID1 partitions on
two physical hard drives, one for */* and another for *swap*.

Partitioning
~~~~~~~~~~~~

Follow the installation steps until you get to the *Partition disks*
step, then:

Select *Manual* as the partition method.

Select the first hard drive, and agree to *"Create a new empty partition
table on this device?"*.

Repeat this step for each drive you wish to be part of the RAID array.

Select the *"FREE SPACE"* on the first drive then select *"Create a new
partition"*.

Next, select the *Size* of the partition. This partition will be the
*swap* partition, and a general rule for swap size is twice that of RAM.
Enter the partition size, then choose *Primary*, then *Beginning*.

    **Note**

    A swap partition size of twice the available RAM capacity may not
    always be desirable, especially on systems with large amounts of
    RAM. Calculating the swap partition size for servers is highly
    dependent on how the system is going to be used.

Select the *"Use as:"* line at the top. By default this is *"Ext4
journaling file system"*, change that to *"physical volume for RAID"*
then *"Done setting up partition"*.

For the */* partition once again select *"Free Space"* on the first
drive then *"Create a new partition"*.

Use the rest of the free space on the drive and choose *Continue*, then
*Primary*.

As with the swap partition, select the *"Use as:"* line at the top,
changing it to *"physical volume for RAID"*. Also select the *"Bootable
flag:"* line to change the value to *"on"*. Then choose *"Done setting
up partition"*.

Repeat steps three through eight for the other disk and partitions.

RAID Configuration
~~~~~~~~~~~~~~~~~~

With the partitions setup the arrays are ready to be configured:

Back in the main "Partition Disks" page, select *"Configure Software
RAID"* at the top.

Select *"yes"* to write the changes to disk.

Choose *"Create MD device"*.

For this example, select *"RAID1"*, but if you are using a different
setup choose the appropriate type (RAID0 RAID1 RAID5).

    **Note**

    In order to use *RAID5* you need at least *three* drives. Using
    RAID0 or RAID1 only *two* drives are required.

Enter the number of active devices *"2"*, or the amount of hard drives
you have, for the array. Then select *"Continue"*.

Next, enter the number of spare devices *"0"* by default, then choose
*"Continue"*.

Choose which partitions to use. Generally they will be sda1, sdb1, sdc1,
etc. The numbers will usually match and the different letters correspond
to different hard drives.

For the *swap* partition choose *sda1* and *sdb1*. Select *"Continue"*
to go to the next step.

Repeat steps *three* through *seven* for the */* partition choosing
*sda2* and *sdb2*.

Once done select *"Finish"*.

Formatting
~~~~~~~~~~

There should now be a list of hard drives and RAID devices. The next
step is to format and set the mount point for the RAID devices. Treat
the RAID device as a local hard drive, format and mount accordingly.

Select *"#1"* under the *"RAID1 device #0"* partition.

Choose *"Use as:"*. Then select *"swap area"*, then *"Done setting up
partition"*.

Next, select *"#1"* under the *"RAID1 device #1"* partition.

Choose *"Use as:"*. Then select *"Ext4 journaling file system"*.

Then select the *"Mount point"* and choose *"/ - the root file system"*.
Change any of the other options as appropriate, then select *"Done
setting up partition"*.

Finally, select *"Finish partitioning and write changes to disk"*.

If you choose to place the root partition on a RAID array, the installer
will then ask if you would like to boot in a *degraded* state. See ? for
further details.

The installation process will then continue normally.

Degraded RAID
~~~~~~~~~~~~~

At some point in the life of the computer a disk failure event may
occur. When this happens, using Software RAID, the operating system will
place the array into what is known as a *degraded* state.

If the array has become degraded, due to the chance of data corruption,
by default Ubuntu Server Edition will boot to *initramfs* after thirty
seconds. Once the initramfs has booted there is a fifteen second prompt
giving you the option to go ahead and boot the system, or attempt manual
recover. Booting to the initramfs prompt may or may not be the desired
behavior, especially if the machine is in a remote location. Booting to
a degraded array can be configured several ways:

-  The dpkg-reconfigure utility can be used to configure the default
   behavior, and during the process you will be queried about additional
   settings related to the array. Such as monitoring, email alerts, etc.
   To reconfigure mdadm enter the following:

   ::

-  The ``dpkg-reconfigure mdadm`` process will change the
   ``/etc/initramfs-tools/conf.d/mdadm`` configuration file. The file
   has the advantage of being able to pre-configure the system's
   behavior, and can also be manually edited:

   ::

       BOOT_DEGRADED=true

       **Note**

       The configuration file can be overridden by using a Kernel
       argument.

-  Using a Kernel argument will allow the system to boot to a degraded
   array as well:

   -  When the server is booting press Shift to open the Grub menu.

   -  Press e to edit your kernel command options.

   -  Press the down arrow to highlight the kernel line.

   -  Add *"bootdegraded=true"* (without the quotes) to the end of the
      line.

   -  Press Ctrlx to boot the system.

Once the system has booted you can either repair the array see ? for
details, or copy important data to another machine due to major hardware
failure.

RAID Maintenance
~~~~~~~~~~~~~~~~

The mdadm utility can be used to view the status of an array, add disks
to an array, remove disks, etc:

-  To view the status of an array, from a terminal prompt enter:

   ::

   The *-D* tells mdadm to display *detailed* information about the
   ``/dev/md0`` device. Replace ``/dev/md0`` with the appropriate RAID
   device.

-  To view the status of a disk in an array:

   ::

   The output if very similar to the ``mdadm -D`` command, adjust
   ``/dev/sda1`` for each disk.

-  If a disk fails and needs to be removed from an array enter:

   ::

   Change ``/dev/md0`` and ``/dev/sda1`` to the appropriate RAID device
   and disk.

-  Similarly, to add a new disk:

   ::

Sometimes a disk can change to a *faulty* state even though there is
nothing physically wrong with the drive. It is usually worthwhile to
remove the drive from the array then re-add it. This will cause the
drive to re-sync with the array. If the drive will not sync with the
array, it is a good indication of hardware failure.

The ``/proc/mdstat`` file also contains useful information about the
system's RAID devices:

::


The following command is great for watching the status of a syncing
drive:

::

Press *Ctrl+c* to stop the watch command.

If you do need to replace a faulty drive, after the drive has been
replaced and synced, grub will need to be installed. To install grub on
the new drive, enter the following:

::

Replace ``/dev/md0`` with the appropriate array device name.

Resources
~~~~~~~~~

The topic of RAID arrays is a complex one due to the plethora of ways
RAID can be configured. Please see the following links for more
information:

-  `Ubuntu Wiki Articles on
   RAID <https://help.ubuntu.com/community/Installation#raid>`__.

-  `Software RAID
   HOWTO <http://www.faqs.org/docs/Linux-HOWTO/Software-RAID-HOWTO.html>`__

-  `Managing RAID on
   Linux <http://oreilly.com/catalog/9781565927308/>`__

Logical Volume Manager (LVM)
----------------------------

Logical Volume Manger, or *LVM*, allows administrators to create
*logical* volumes out of one or multiple physical hard disks. LVM
volumes can be created on both software RAID partitions and standard
partitions residing on a single disk. Volumes can also be extended,
giving greater flexibility to systems as requirements change.

Overview
~~~~~~~~

A side effect of LVM's power and flexibility is a greater degree of
complication. Before diving into the LVM installation process, it is
best to get familiar with some terms.

-  *Physical Volume (PV):* physical hard disk, disk partition or
   software RAID partition formatted as LVM PV.

-  *Volume Group (VG):* is made from one or more physical volumes. A VG
   can can be extended by adding more PVs. A VG is like a virtual disk
   drive, from which one or more logical volumes are carved.

-  *Logical Volume (LV):* is similar to a partition in a non-LVM system.
   A LV is formatted with the desired file system (EXT3, XFS, JFS, etc),
   it is then available for mounting and data storage.

Installation
~~~~~~~~~~~~

As an example this section covers installing Ubuntu Server Edition with
``/srv`` mounted on a LVM volume. During the initial install only one
Physical Volume (PV) will be part of the Volume Group (VG). Another PV
will be added after install to demonstrate how a VG can be extended.

There are several installation options for LVM, *"Guided - use the
entire disk and setup LVM"* which will also allow you to assign a
portion of the available space to LVM, *"Guided - use entire and setup
encrypted LVM"*, or *Manually* setup the partitions and configure LVM.
At this time the only way to configure a system with both LVM and
standard partitions, during installation, is to use the Manual approach.

Follow the installation steps until you get to the *Partition disks*
step, then:

At the *"Partition Disks* screen choose *"Manual"*.

Select the hard disk and on the next screen choose "yes" to *"Create a
new empty partition table on this device"*.

Next, create standard */boot*, *swap*, and */* partitions with whichever
filesystem you prefer.

For the LVM */srv*, create a new *Logical* partition. Then change *"Use
as"* to *"physical volume for LVM"* then *"Done setting up the
partition"*.

Now select *"Configure the Logical Volume Manager"* at the top, and
choose *"Yes"* to write the changes to disk.

For the *"LVM configuration action"* on the next screen, choose *"Create
volume group"*. Enter a name for the VG such as *vg01*, or something
more descriptive. After entering a name, select the partition configured
for LVM, and choose *"Continue"*.

Back at the *"LVM configuration action"* screen, select *"Create logical
volume"*. Select the newly created volume group, and enter a name for
the new LV, for example *srv* since that is the intended mount point.
Then choose a size, which may be the full partition because it can
always be extended later. Choose *"Finish"* and you should be back at
the main *"Partition Disks"* screen.

Now add a filesystem to the new LVM. Select the partition under *"LVM VG
vg01, LV srv"*, or whatever name you have chosen, the choose *Use as*.
Setup a file system as normal selecting */srv* as the mount point. Once
done, select *"Done setting up the partition"*.

Finally, select *"Finish partitioning and write changes to disk"*. Then
confirm the changes and continue with the rest of the installation.

There are some useful utilities to view information about LVM:

-  *pvdisplay:* shows information about Physical Volumes.

-  *vgdisplay:* shows information about Volume Groups.

-  *lvdisplay:* shows information about Logical Volumes.

Extending Volume Groups
~~~~~~~~~~~~~~~~~~~~~~~

Continuing with *srv* as an LVM volume example, this section covers
adding a second hard disk, creating a Physical Volume (PV), adding it to
the volume group (VG), extending the logical volume ``srv`` and finally
extending the filesystem. This example assumes a second hard disk has
been added to the system. In this example, this hard disk will be named
``/dev/sdb`` and we will use the entire disk as a physical volume (you
could choose to create partitions and use them as different physical
volumes)

    **Warning**

    Make sure you don't already have an existing ``/dev/sdb`` before
    issuing the commands below. You could lose some data if you issue
    those commands on a non-empty disk.

First, create the physical volume, in a terminal execute:

::


                    

Now extend the Volume Group (VG):

::

Use vgdisplay to find out the free physical extents - Free PE / size
(the size you can allocate). We will assume a free size of 511 PE
(equivalent to 2GB with a PE size of 4MB) and we will use the whole free
space available. Use your own PE and/or free space.

The Logical Volume (LV) can now be extended by different methods, we
will only see how to use the PE to extend the LV:

::

The *-l* option allows the LV to be extended using PE. The *-L* option
allows the LV to be extended using Meg, Gig, Tera, etc bytes.

Even though you are supposed to be able to *expand* an ext3 or ext4
filesystem without unmounting it first, it may be a good practice to
unmount it anyway and check the filesystem, so that you don't mess up
the day you want to reduce a logical volume (in that case unmounting
first is compulsory).

The following commands are for an *EXT3* or *EXT4* filesystem. If you
are using another filesystem there may be other utilities available.

::


The *-f* option of e2fsck forces checking even if the system seems
clean.

Finally, resize the filesystem:

::

Now mount the partition and check its size.

::

Resources
~~~~~~~~~

-  See the `Ubuntu Wiki LVM
   Articles <https://help.ubuntu.com/community/Installation#lvm>`__.

-  See the `LVM HOWTO <http://tldp.org/HOWTO/LVM-HOWTO/index.html>`__
   for more information.

-  Another good article is `Managing Disk Space with
   LVM <http://www.linuxdevcenter.com/pub/a/linux/2006/04/27/managing-disk-space-with-lvm.html>`__
   on O'Reilly's linuxdevcenter.com site.

-  For more information on fdisk see the `fdisk man
   page <http://manpages.ubuntu.com/manpages/&distro-short-codename;/en/man8/fdisk.8.html>`__.

Kernel Crash Dump
=================

Introduction
------------

A Kernel Crash Dump refers to a portion of the contents of volatile
memory (RAM) that is copied to disk whenever the execution of the kernel
is disrupted. The following events can cause a kernel disruption :

-  Kernel Panic

-  Non Maskable Interrupts (NMI)

-  Machine Check Exceptions (MCE)

-  Hardware failure

-  Manual intervention

For some of those events (panic, NMI) the kernel will react
automatically and trigger the crash dump mechanism through *kexec*. In
other situations a manual intervention is required in order to capture
the memory. Whenever one of the above events occurs, it is important to
find out the root cause in order to prevent it from happening again. The
cause can be determined by inspecting the copied memory contents.

Kernel Crash Dump Mechanism
---------------------------

When a kernel panic occurs, the kernel relies on the *kexec* mechanism
to quickly reboot a new instance of the kernel in a pre-reserved section
of memory that had been allocated when the system booted (see below).
This permits the existing memory area to remain untouched in order to
safely copy its contents to storage.

Installation
------------

The kernel crash dump utility is installed with the following command:

::

A reboot is then needed.

Configuration
-------------

No further configuration is required in order to have the kernel dump
mechanism enabled.

Verification
------------

To confirm that the kernel dump mechanism is enabled, there are a few
things to verify. First, confirm that the *crashkernel* boot parameter
is present (note: The following line has been split into two to fit the
format of this document:

::


The *crashkernel* parameter has the following syntax:

::

    crashkernel=<range1>:<size1>[,<range2>:<size2>,...][@offset]
        range=start-[end] 'start' is inclusive and 'end' is exclusive.
            

So for the crashkernel parameter found in ``/proc/cmdline`` we would
have :

::

    crashkernel=384M-2G:64M,2G-:128M

The above value means:

-  if the RAM is smaller than 384M, then don't reserve anything (this is
   the "rescue" case)

-  if the RAM size is between 386M and 2G (exclusive), then reserve 64M

-  if the RAM size is larger than 2G, then reserve 128M

Second, verify that the kernel has reserved the requested memory area
for the kdump kernel by doing:

::


Testing the Crash Dump Mechanism
--------------------------------

    **Warning**

    Testing the Crash Dump Mechanism will cause *a system reboot.* In
    certain situations, this can cause data loss if the system is under
    heavy load. If you want to test the mechanism, make sure that the
    system is idle or under very light load.

Verify that the *SysRQ* mechanism is enabled by looking at the value of
the ``/proc/sys/kernel/sysrq`` kernel parameter :

::

If a value of *0* is returned the feature is disabled. Enable it with
the following command :

::

Once this is done, you must become root, as just using ``sudo`` will not
be sufficient. As the *root* user, you will have to issue the command
``echo c > /proc/sysrq-trigger``. If you are using a network connection,
you will lose contact with the system. This is why it is better to do
the test while being connected to the system console. This has the
advantage of making the kernel dump process visible.

A typical test output should look like the following :

::


    [sudo] password for ubuntu: 
    # 
    [   31.659002] SysRq : Trigger a crash
    [   31.659749] BUG: unable to handle kernel NULL pointer dereference at           (null)
    [   31.662668] IP: [<ffffffff8139f166>] sysrq_handle_crash+0x16/0x20
    [   31.662668] PGD 3bfb9067 PUD 368a7067 PMD 0 
    [   31.662668] Oops: 0002 [#1] SMP 
    [   31.662668] CPU 1 
    ....

The rest of the output is truncated, but you should see the system
rebooting and somewhere in the log, you will see the following line :

::

    Begin: Saving vmcore from kernel crash ...

Once completed, the system will reboot to its normal operational mode.
You will then find Kernel Crash Dump file in the ``/var/crash``
directory :

::


    linux-image-3.0.0-12-server.0.crash

Resources
---------

Kernel Crash Dump is a vast topic that requires good knowledge of the
linux kernel. You can find more information on the topic here :

-  `Kdump kernel
   documentation <http://www.kernel.org/doc/Documentation/kdump/kdump.txt>`__.

-  `The crash tool <http://people.redhat.com/~anderson/>`__

-  `Analyzing Linux Kernel
   Crash <http://www.dedoimedo.com/computers/crash-analyze.html>`__
   (Based on Fedora, it still gives a good walkthrough of kernel dump
   analysis)


