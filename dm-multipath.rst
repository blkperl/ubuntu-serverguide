DMM
===

Device Mapper Multipathing
==========================

Device mapper multipathing (DM-Multipath) allows you to configure
multiple I/O paths between server nodes and storage arrays into a single
device. These I/O paths are physical SAN connections that can include
separate cables, switches, and controllers. Multipathing aggregates the
I/O paths, creating a new device that consists of the aggregated paths.
This chapter provides a summary of the features of DM-Multipath that are
new for the initial release of Ubuntu Server 12.04. Following that, this
chapter provides a high-level overview of DM Multipath and its
components, as well as an overview of DM-Multipath setup.

New and Changed Features for Ubuntu Server 12.04
------------------------------------------------

Migrated from multipath-0.4.8 to multipath-0.4.9

Migration from 0.4.8
~~~~~~~~~~~~~~~~~~~~

The priority checkers are no longer run as standalone binaries, but as
shared libraries. The key value name for this feature has also slightly
changed. Copy the attribute named **prio\_callout** to **prio**, also
modify the argument the name of the priority checker, a system path is
no longer necessary. Example conversion:

::

    device {
            vendor "NEC"
            product "DISK ARRAY"
            prio_callout mpath_prio_alua /dev/%n
            prio    alua
    }

See Table ? for a complete listing

+--------------------------------------------------+--------------------------+
| v0.4.8                                           | v0.4.9                   |
+==================================================+==========================+
| **prio\_callout mpath\_prio\_emc /dev/%n**       | **prio emc**             |
+--------------------------------------------------+--------------------------+
| **prio\_callout mpath\_prio\_alua /dev/%n**      | **prio alua**            |
+--------------------------------------------------+--------------------------+
| **prio\_callout mpath\_prio\_netapp /dev/%n**    | **prio netapp**          |
+--------------------------------------------------+--------------------------+
| **prio\_callout mpath\_prio\_rdac /dev/%n**      | **prio rdac**            |
+--------------------------------------------------+--------------------------+
| **prio\_callout mpath\_prio\_hp\_sw /dev/%n**    | **prio hp\_sw**          |
+--------------------------------------------------+--------------------------+
| **prio\_callout mpath\_prio\_hds\_modular %b**   | **prio hds**             |
+--------------------------------------------------+--------------------------+

Table: Priority Checker Conversion

Since the multipath config file parser essentially parses all key/value
pairs it finds and then makes use of them, it is safe for both
**prio\_callout** and **prio** to coexist and is recommended that the
**prio** attribute be inserted before beginning migration. After which
you can safely delete the legacy **prio\_calliout** attribute without
interrupting service.

Overview
--------

DM-Multipath can be used to provide:

-  *Redundancy* DM-Multipath can provide failover in an active/passive
   configuration. In an active/passive configuration, only half the
   paths are used at any time for I/O. If any element of an I/O path
   (the cable, switch, or controller) fails, DM-Multipath switches to an
   alternate path.

-  *Improved Performance* Performance DM-Multipath can be configured in
   active/active mode, where I/O is spread over the paths in a
   round-robin fashion. In some configurations, DM-Multipath can detect
   loading on the I/O paths and dynamically re-balance the load.

Storage Array Overview
----------------------

By default, DM-Multipath includes support for the most common storage
arrays that support DM-Multipath. The supported devices can be found in
the multipath.conf.defaults file. If your storage array supports
DM-Multipath and is not configured by default in this file, you may need
to add them to the DM-Multipath configuration file, multipath.conf. For
information on the DM-Multipath configuration file, see Section, `The
DM-Multipath Configuration
File <#multipath-dm-multipath-config-file>`__. Some storage arrays
require special handling of I/O errors and path switching. These require
separate hardware handler kernel modules.

DMM components
--------------

Table `DM-Multipath Components <#multipath-components-table>`__
describes the components of the DM-Multipath package.

DMM Setup Overview
------------------

DM-Multipath includes compiled-in default settings that are suitable for
common multipath configurations. Setting up DM-multipath is often a
simple procedure. The basic procedure for configuring your system with
DM-Multipath is as follows:

1. Install the **multipath-tools** and **multipath-tools-boot** packages

2. Create an empty config file, MULTIPATH-CONF-FULL, that re-defines the
   `following <#multipath-skel-config>`__

3. If necessary, edit the **multipath.conf** configuration file to
   modify default values and save the updated file.

4. Start the multipath daemon

5. Update initial ramdisk

For detailed setup instructions for multipath configuration see Section,
`Setting Up DM-Multipath <#multipath-setup-overview>`__.

Multipath Devices
=================

Without DM-Multipath, each path from a server node to a storage
controller is treated by the system as a separate device, even when the
I/O path connects the same server node to the same storage controller.
DM-Multipath provides a way of organizing the I/O paths logically, by
creating a single multipath device on top of the underlying devices.

Multipath Device Identifiers
----------------------------

Each multipath device has a World Wide Identifier (WWID), which is
guaranteed to be globally unique and unchanging. By default, the name of
a multipath device is set to its WWID. Alternately, you can set the
**`user\_friendly\_names <#attribute-user_friendly_names>`__**\ option
in the multipath configuration file, which causes DM-Multipath to use a
node-unique alias of the form **mpathn** as the name. For example, a
node with two HBAs attached to a storage controller with two ports via a
single unzoned FC switch sees four devices: **/dev/sda**, **/dev/sdb**,
**/dev/sdc**, and **/dev/sdd**. DM-Multipath creates a single device
with a unique WWID that reroutes I/O to those four underlying devices
according to the multipath configuration. When the
**`user\_friendly\_names <#attribute-user_friendly_names>`__**
configuration option is set to **yes**, the name of the multipath device
is set to **mpathn**. When new devices are brought under the control of
DM-Multipath, the new devices may be seen in two different places under
the **/dev** directory: **/dev/mapper/mpathn** and **/dev/dm-n**.

-  The devices in **/dev/mapper** are created early in the boot process.
   Use these devices to access the multipathed devices, for example when
   creating logical volumes.

-  Any devices of the form **/dev/dm-n** are for internal use only and
   should never be used.

For information on the multipath configuration defaults, including the
**`user\_friendly\_names <#attribute-user_friendly_names>`__**
configuration option, see Section , `Configuration File
Defaults <#multipath-config-defaults>`__. You can also set the name of a
multipath device to a name of your choosing by using the
**`alias <#attribute-alias>`__** option in the **multipaths** section of
the multipath configuration file. For information on the **multipaths**
section of the multipath configuration file, see Section, `Multipaths
Device Configuration Attributes <#multipath-config-multipath>`__.

Consistent Multipath Device Names in a Cluster
----------------------------------------------

When the **user\_friendly\_names** configuration option is set to yes,
the name of the multipath device is unique to a node, but it is not
guaranteed to be the same on all nodes using the multipath device.
Similarly, if you set the **alias** option for a device in the
**multipaths** section of the MULTIPATH-CONF configuration file, the
name is not automatically consistent across all nodes in the cluster.
This should not cause any difficulties if you use LVM to create logical
devices from the multipath device, but if you require that your
multipath device names be consistent in every node it is recommended
that you leave the **user\_friendly\_names** option set to **no** and
that you not configure aliases for the devices. By default, if you do
not set **user\_friendly\_names** to yes or configure an alias for a
device, a device name will be the WWID for the device, which is always
the same. If you want the system-defined user-friendly names to be
consistent across all nodes in the cluster, however, you can follow this
procedure:

1. Set up all of the multipath devices on one machine.

2. Disable all of your multipath devices on your other machines by
   running the following commands:

   ::

       # service multipath-tools stop
       # multipath -F

3. Copy the ``/etc/multipath/bindings`` file from the first machine to
   all the other machines in the cluster.

4. Re-enable the multipathd daemon on all the other machines in the
   cluster by running the following command:

   ::

       # service multipath-tools start

If you add a new device, you will need to repeat this process.

Similarly, if you configure an alias for a device that you would like to
be consistent across the nodes in the cluster, you should ensure that
the ``/etc/multipath.conf`` file is the same for each node in the
cluster by following the same procedure:

1. Configure the aliases for the multipath devices in the in the
   ``multipath.conf`` file on one machine.

2. Disable all of your multipath devices on your other machines by
   running the following commands:

   ::

       # service multipath-tools stop
       # multipath -F

3. Copy the MULTIPATH-CONF file from the first machine to all the other
   machines in the cluster.

4. Re-enable the multipathd daemon on all the other machines in the
   cluster by running the following command:

   ::

       # service multipath-tools start

When you add a new device you will need to repeat this process.

Multipath Device attributes
---------------------------

In addition to the **user\_friendly\_names** and **alias** options, a
multipath device has numerous attributes. You can modify these
attributes for a specific multipath device by creating an entry for that
device in the **multipaths** section of the **multipath** configuration
file. For information on the **multipaths** section of the multipath
configuration file, see Section,
"`#multipath-config-multipath <#multipath-config-multipath>`__\ ".

Multipath Devices in Logical Volumes
------------------------------------

After creating multipath devices, you can use the multipath device names
just as you would use a physical device name when creating an LVM
physical volume. For example, if /dev/mapper/mpatha is the name of a
multipath device, the following command will mark /dev/mapper/mpatha as
a physical volume.

::

    # pvcreate /dev/mapper/mpatha

You can use the resulting LVM physical device when you create an LVM
volume group just as you would use any other LVM physical device.

    **Note**

    If you attempt to create an LVM physical volume on a whole device on
    which you have configured partitions, the pvcreate command will
    fail.

When you create an LVM logical volume that uses active/passive multipath
arrays as the underlying physical devices, you should include filters in
the **lvm.conf** to exclude the disks that underlie the multipath
devices. This is because if the array automatically changes the active
path to the passive path when it receives I/O, multipath will failover
and failback whenever LVM scans the passive path if these devices are
not filtered. For active/passive arrays that require a command to make
the passive path active, LVM prints a warning message when this occurs.
To filter all SCSI devices in the LVM configuration file (lvm.conf),
include the following filter in the devices section of the file.

::

    filter = [ "r/block/", "r/disk/", "r/sd.*/", "a/.*/" ]

After updating ``/etc/lvm.conf``, it's necessary to update the
**initrd** so that this file will be copied there, where the filter
matters the most, during boot. Perform:

::

    update-initramfs -u -k all

    **Note**

    Every time either ``/etc/lvm.conf`` or ``/etc/multipath.conf`` is
    updated, the initrd should be rebuilt to reflect these changes. This
    is imperative when blacklists and filters are necessary to maintain
    a stable storage configuration.

Setting up DMM Overview
=======================

This section provides step-by-step example procedures for configuring
DMM. It includes the following procedures:

-  Basic DMM setup

-  Ignoring local disks

-  Adding more devices to the configuration file

Setting Up DMM
--------------

Before setting up DMM on your system, ensure that your system has been
updated and includes the **multipath-tools** package. If boot from SAN
is desired, then the **multipath-tools-boot** package is also required.

A basic **/etc/multipath.conf** need not even exist, when **multpath**
is run without an accompanying ``/etc/multipath.conf``, it draws from
it's internal database to find a suitable configuration, it also draws
from it's internal blacklist. If after running **multipath -ll** without
a config file, no multipaths are discovered. One must proceed to
increase the verbosity to discover why a multipath was not created.
Consider referencing the SAN vendor's documentation, the multipath
example config files found in
``/usr/share/doc/multipath-tools/examples``, and the live multipathd
database:

::

    # echo 'show config' | multipathd -k > multipath.conf-live

    **Note**

    To work around a quirk in multipathd, when an
    ``/etc/multipath.conf`` doesn't exist, the previous command will
    return nothing, as it is the result of a *merge* between the
    ``/etc/multipath.conf`` and the database in memory. To remedy this,
    either define an empty ``/etc/multipath.conf``, by using **touch**,
    or create one that redefines a default value like:

    ::

        defaults {
                user_friendly_names no
        }

    and restart multipathd:

    ::

        # service multipath-tools restart

    Now the "show config" command will return the live database.

Installing with Multipath Support
---------------------------------

To enable `multipath support during
installation <http://wiki.debian.org/DebianInstaller/MultipathSupport>`__
use

::

    install disk-detect/multipath/enable=true

at the installer prompt. If multipath devices are found these will show
up as **/dev/mapper/mpath<X>** during installation.

Ignoring Local Disks When Generating Multipath Devices
------------------------------------------------------

Some machines have local SCSI cards for their internal disks.
DM-Multipath is not recommended for these devices. The following
procedure shows how to modify the multipath configuration file to ignore
the local disks when configuring multipath.

1. Determine which disks are the internal disks and mark them as the
   ones to blacklist. In this example, **``/dev/sda``** is the internal
   disk. Note that as originally configured in the default multipath
   configuration file, executing the **multipath -v2** shows the local
   disk, **/dev/sda**, in the multipath map. For further information on
   the **multipath** command output, see Section `Multipath Command
   Output <#multipath-command-output>`__.

   ::

       # multipath -v2
       create: SIBM-ESXSST336732LC____F3ET0EP0Q000072428BX1 undef WINSYS,SF2372
       size=33 GB features="0" hwhandler="0" wp=undef
       `-+- policy='round-robin 0' prio=1 status=undef
         |- 0:0:0:0 sda 8:0  [--------- 

       device-mapper ioctl cmd 9 failed: Invalid argument
       device-mapper ioctl cmd 14 failed: No such device or address
       create: 3600a0b80001327d80000006d43621677 undef WINSYS,SF2372
       size=12G features='0' hwhandler='0' wp=undef
       `-+- policy='round-robin 0' prio=1 status=undef
         |- 2:0:0:0 sdb 8:16  undef ready  running
           `- 3:0:0:0 sdf 8:80 undef ready  running

       create: 3600a0b80001327510000009a436215ec undef WINSYS,SF2372
       size=12G features='0' hwhandler='0' wp=undef
       `-+- policy='round-robin 0' prio=1 status=undef
         |- 2:0:0:1 sdc 8:32 undef ready  running
           `- 3:0:0:1 sdg 8:96 undef ready  running

       create: 3600a0b80001327d800000070436216b3 undef WINSYS,SF2372
       size=12G features='0' hwhandler='0' wp=undef
       `-+- policy='round-robin 0' prio=1 status=undef
         |- 2:0:0:2 sdd 8:48 undef ready  running
           `- 3:0:0:2 sdg 8:112 undef ready  running

       create: 3600a0b80001327510000009b4362163e undef WINSYS,SF2372
       size=12G features='0' hwhandler='0' wp=undef
       `-+- policy='round-robin 0' prio=1 status=undef
         |- 2:0:0:3 sdd 8:64 undef ready  running
           `- 3:0:0:3 sdg 8:128 undef ready  running

2. In order to prevent the device mapper from mapping **/dev/sda** in
   its multipath maps, edit the blacklist section of the
   MULTIPATH-CONF-FULL file to include this device. Although you could
   blacklist the **sda** device using a **devnode** type, that would not
   be safe procedure since **/dev/sda** is not guaranteed to be the same
   on reboot. To blacklist individual devices, you can blacklist using
   the WWID of that device. Note that in the output to the **multipath
   -v2** command, the WWID of the ``/dev/sda`` device is
   SIBM-ESXSST336732LC\_\_\_\_F3ET0EP0Q000072428BX1. To blacklist this
   device, include the following in the MULTIPATH-CONF-FULL file.

   ::

       blacklist {
             wwid SIBM-ESXSST336732LC____F3ET0EP0Q000072428BX1
       }

3. After you have updated the MULTIPATH-CONF-FULL file, you must
   manually tell the **multipathd** daemon to reload the file. The
   following command reloads the updated MULTIPATH-CONF-FULL file.

   ::

       # service multipath-tools reload

4. Run the following command to remove the multipath device:

   ::

       # multipath -f SIBM-ESXSST336732LC____F3ET0EP0Q000072428BX1

5. To check whether the device removal worked, you can run the
   ``multipath -ll`` command to display the current multipath
   configuration. For information on the ``multipath
             -ll`` command, see Section `Multipath Queries with
   multipath Command <#multipath-queries-and-commands>`__. To check that
   the blacklisted device was not added back, you can run the multipath
   command, as in the following example. The multipath command defaults
   to a verbosity level of **v2** if you do not specify a **-v** option.

   ::

       # multipath

       create: 3600a0b80001327d80000006d43621677 undef WINSYS,SF2372
       size=12G features='0' hwhandler='0' wp=undef
       `-+- policy='round-robin 0' prio=1 status=undef
         |- 2:0:0:0 sdb 8:16  undef ready  running
           `- 3:0:0:0 sdf 8:80 undef ready  running

       create: 3600a0b80001327510000009a436215ec undef WINSYS,SF2372
       size=12G features='0' hwhandler='0' wp=undef
       `-+- policy='round-robin 0' prio=1 status=undef
         |- 2:0:0:1 sdc 8:32 undef ready  running
           `- 3:0:0:1 sdg 8:96 undef ready  running

       create: 3600a0b80001327d800000070436216b3 undef WINSYS,SF2372
       size=12G features='0' hwhandler='0' wp=undef
       `-+- policy='round-robin 0' prio=1 status=undef
         |- 2:0:0:2 sdd 8:48 undef ready  running
           `- 3:0:0:2 sdg 8:112 undef ready  running

       create: 3600a0b80001327510000009b4362163e undef WINSYS,SF2372
       size=12G features='0' hwhandler='0' wp=undef
       `-+- policy='round-robin 0' prio=1 status=undef
         |- 2:0:0:3 sdd 8:64 undef ready  running
           `- 3:0:0:3 sdg 8:128 undef ready  running

Configuring Storage Devices
---------------------------

By default, DM-Multipath includes support for the most common storage
arrays that support DM-Multipath. The default configuration values,
including supported devices, can be found in the
``multipath.conf.defaults`` file.

If you need to add a storage device that is not supported by default as
a known multipath device, edit the MULTIPATH-CONF-FULL file and insert
the appropriate device information.

For example, to add information about the HP Open-V series the entry
looks like this, where **%n** is the device name:

::

    devices {
         device {
                vendor "HP"
                product "OPEN-V."
                getuid_callout "/lib/udev/scsi_id --whitelisted --device=/dev/%n"
         }
    }

For more information on the devices section of the configuration file,
see Section ?.

The DMM Configuration File
==========================

By default, DM-Multipath provides configuration values for the most
common uses of multipathing. In addition, DM-Multipath includes support
for the most common storage arrays that support DM-Multipath. The
default configuration values and the supported devices can be found in
the ``multipath.conf.defaults`` file.

You can override the default configuration values for DM-Multipath by
editing the MULTIPATH-CONF-FULL configuration file. If necessary, you
can also add a storage array that is not supported by default to the
configuration file. This chapter provides information on parsing and
modifying the MULTIPATH-CONF file. It contains sections on the following
topics:

-  ?

-  ?

-  ?

-  ?

-  ?

In the multipath configuration file, you need to specify only the
sections that you need for your configuration, or that you wish to
change from the default values specified in the
``multipath.conf.defaults`` file. If there are sections of the file that
are not relevant to your environment or for which you do not need to
override the default values, you can leave them commented out, as they
are in the initial file.

The configuration file allows regular expression description syntax.

An annotated version of the configuration file can be found in ````.

Configuration File Overview
---------------------------

The multipath configuration file is divided into the following sections:

**blacklist**
    Listing of specific devices that will not be considered for
    multipath.

**blacklist\_exceptions**
    Listing of multipath candidates that would otherwise be blacklisted
    according to the parameters of the blacklist section.

**defaults**
    General default settings for DM-Multipath.

**multipath**
    Settings for the characteristics of individual multipath devices.
    These values overwrite what is specified in the **defaults** and
    **devices** sections of the configuration file.

**devices**
    Settings for the individual storage controllers. These values
    overwrite what is specified in the **defaults** section of the
    configuration file. If you are using a storage array that is not
    supported by default, you may need to create a devices subsection
    for your array.

When the system determines the attributes of a multipath device, first
it checks the multipath settings, then the per devices settings, then
the multipath system defaults.

Configuration File Blacklist
----------------------------

The blacklist section of the multipath configuration file specifies the
devices that will not be used when the system configures multipath
devices. Devices that are blacklisted will not be grouped into a
multipath device.

-  If you do need to blacklist devices, you can do so according to the
   following criteria:

   -  By WWID, as described ?

   -  By device name, as described in ?

   -  By device type, as described in ?

   By default, a variety of device types are blacklisted, even after you
   comment out the initial blacklist section of the configuration file.
   For information, see ?

Blacklisting By WWID
~~~~~~~~~~~~~~~~~~~~

You can specify individual devices to blacklist by their World-Wide
IDentification with a **wwid** entry in the **blacklist** section of the
configuration file.

The following example shows the lines in the configuration file that
would blacklist a device with a WWID of 26353900f02796769.

::

    blacklist {
           wwid 26353900f02796769
    }

Blacklisting By Device Name
~~~~~~~~~~~~~~~~~~~~~~~~~~~

You can blacklist device types by device name so that they will not be
grouped into a multipath device by specifying a **devnode** entry in the
**blacklist** section of the configuration file.

The following example shows the lines in the configuration file that
would blacklist all SCSI devices, since it blacklists all sd\* devices.

::

    blacklist {
           devnode "^sd[a-z]"
    }

You can use a **devnode** entry in the **blacklist** section of the
configuration file to specify individual devices to blacklist rather
than all devices of a specific type. This is not recommended, however,
since unless it is statically mapped by udev rules, there is no
guarantee that a specific device will have the same name on reboot. For
example, a device name could change from ``/dev/sda`` to ``/dev/sdb`` on
reboot.

By default, the following **devnode** entries are compiled in the
default blacklist; the devices that these entries blacklist do not
generally support DM-Multipath. To enable multipathing on any of these
devices, you would need to specify them in the **blacklist\_exceptions**
section of the configuration file, as described in ?

::

    blacklist {
           devnode "^(ram|raw|loop|fd|md|dm-|sr|scd|st)[0-9]*"
           devnode "^hd[a-z]"
    }

Blacklisting By Device Type
~~~~~~~~~~~~~~~~~~~~~~~~~~~

You can specify specific device types in the **blacklist** section of
the configuration file with a device section. The following example
blacklists all IBM DS4200 and HP devices.

::

    blacklist {
           device {
                   vendor  "IBM"
                   product "3S42"       #DS4200 Product 10
           }
           device {
                   vendor  "HP"
                   product "*"
           }
    }

Blacklist Exceptions
~~~~~~~~~~~~~~~~~~~~

You can use the **blacklist\_exceptions** section of the configuration
file to enable multipathing on devices that have been blacklisted by
default.

For example, if you have a large number of devices and want to multipath
only one of them (with the WWID of 3600d0230000000000e13955cc3757803),
instead of individually blacklisting each of the devices except the one
you want, you could instead blacklist all of them, and then allow only
the one you want by adding the following lines to the
MULTIPATH-CONF-FULL file.

::

    blacklist {
            wwid "*"
    }

    blacklist_exceptions {
            wwid "3600d0230000000000e13955cc3757803"
    }

When specifying devices in the **blacklist\_exceptions** section of the
configuration file, you must specify the exceptions in the same way they
were specified in the **blacklist**. For example, a WWID exception will
not apply to devices specified by a **devnode** blacklist entry, even if
the blacklisted device is associated with that WWID. Similarly, devnode
exceptions apply only to devnode entries, and device exceptions apply
only to device entries.

Configuration File Defaults
---------------------------

The MULTIPATH-CONF-FULL configuration file includes a **defaults**
section that sets the **user\_friendly\_names** parameter to **yes**, as
follows.

::

    defaults {
            user_friendly_names yes
    }

This overwrites the default value of the **user\_friendly\_names**
parameter.

The configuration file includes a template of configuration defaults.
This section is commented out, as follows.

::

    #defaults {
    #       udev_dir                /dev
    #       polling_interval        5
    #       selector                "round-robin 0"
    #       path_grouping_policy    failover
    #       getuid_callout          "/lib/dev/scsi_id --whitelisted --device=/dev/%n"
    #   prio            const
    #   path_checker        directio
    #   rr_min_io       1000
    #   rr_weight       uniform
    #   failback        manual
    #   no_path_retry       fail
    #   user_friendly_names no
    #}

To overwrite the default value for any of the configuration parameters,
you can copy the relevant line from this template into the **defaults**
section and uncomment it. For example, to overwrite the
**path\_grouping\_policy** parameter so that it is **multibus** rather
than the default value of **failover**, copy the appropriate line from
the template to the initial **defaults** section of the configuration
file, and uncomment it, as follows.

::

    defaults {
            user_friendly_names     yes
            path_grouping_policy    multibus
    }

Table ? describes the attributes that are set in the **defaults**
section of the ``multipath.conf`` configuration file. These values are
used by DM-Multipath unless they are overwritten by the attributes
specified in the **devices** and **multipaths** sections of the
MULTIPATH-CONF file.

Configuration File Multipath Attributes
---------------------------------------

Table ? shows the attributes that you can set in the **multipaths**
section of the ``multipath.conf`` configuration file for each specific
multipath device. These attributes apply only to the one specified
multipath. These defaults are used by DM-Multipath and override
attributes set in the **defaults** and **devices** sections of the
multipath.conf file.

In addition, the following parameters may be overridden in this
**multipath** section

-  ```path_grouping_policy`` <#attribute-path_grouping_policy>`__

-  ```path_selector`` <#attribute-path_selector>`__

-  ```failback`` <#attribute-failback>`__

-  ```prio`` <#attribute-prio>`__

-  ```prio_args`` <#attribute-prio_args>`__

-  ```no_path_retry`` <#attribute-no_path_retry>`__

-  ```rr_min_io`` <#attribute-rr_min_io>`__

-  ```rr_weight`` <#attribute-rr_weight>`__

-  ```flush_on_last_del`` <#attribute-flush_on_last_del>`__

The following example shows multipath attributes specified in the
configuration file for two specific multipath devices. The first device
has a WWID of 3600508b4000156d70001200000b0000 and a symbolic name of
yellow.

The second multipath device in the example has a WWID of
1DEC\_\_\_\_\_321816758474 and a symbolic name of red. In this example,
the `rr\_weight <#attribute-rr_weight>`__ attributes are set to
priorities.

::

    multipaths {
           multipath {
                  wwid                  3600508b4000156d70001200000b0000
                  alias                 yellow
                  path_grouping_policy  multibus
                  path_selector         "round-robin 0"
                  failback              manual
                  rr_weight             priorities
                  no_path_retry         5
           }
           multipath {
                  wwid                  1DEC_____321816758474
                  alias                 red
                  rr_weight             priorities
            }
    }

Configuration File Devices
--------------------------

Table ? shows the attributes that you can set for each individual
storage device in the devices section of the multipath.conf
configuration file. These attributes are used by DM-Multipath unless
they are overwritten by the attributes specified in the **multipaths**
section of the ``multipath.conf`` file for paths that contain the
device. These attributes override the attributes set in the **defaults**
section of the ``multipath.conf`` file.

Many devices that support multipathing are included by default in a
multipath configuration. The values for the devices that are supported
by default are listed in the ``multipath.conf.defaults`` file. You
probably will not need to modify the values for these devices, but if
you do you can overwrite the default values by including an entry in the
configuration file for the device that overwrites those values. You can
copy the device configuration defaults from the
``multipath.conf.annotated.gz`` or if you wish to have a brief config
file, ``multipath.conf.synthetic`` file for the device and override the
values that you want to change.

To add a device to this section of the configuration file that is not
configured automatically by default, you must set the **vendor** and
**product** parameters. You can find these values by looking at
**/sys/block/device\_name/device/vendor** and
**/sys/block/device\_name/device/model** where device\_name is the
device to be multipathed, as in the following example:

::

    # cat /sys/block/sda/device/vendor
    WINSYS  
    # cat /sys/block/sda/device/model
    SF2372

The additional parameters to specify depend on your specific device. If
the device is active/active, you will usually not need to set additional
parameters. You may want to set
`path\_grouping\_policy <#attribute-path_grouping_policy>`__ to
**multibus**. Other parameters you may need to set are
`no\_path\_retry <#attribute-no_path_retry>`__ and
`rr\_min\_io <#attribute-rr_min_io>`__, as described in Table ?.

If the device is active/passive, but it automatically switches paths
with I/O to the passive path, you need to change the checker function to
one that does not send I/O to the path to test if it is working
(otherwise, your device will keep failing over). This almost always
means that you set the `path\_checker <#attribute-path_checker>`__ to
**tur**; this works for all SCSI devices that support the Test Unit
Ready command, which most do.

If the device needs a special command to switch paths, then configuring
this device for multipath requires a hardware handler kernel module. The
current available hardware handler is emc. If this is not sufficient for
your device, you may not be able to configure the device for multipath.

In addition, the following parameters may be overridden in this
**device** section

-  ```path_grouping_policy`` <#attribute-path_grouping_policy>`__

-  ```getuid_callout`` <#attribute-getuid_callout>`__

-  ```path_selector`` <#attribute-path_selector>`__

-  ```path_checker`` <#attribute-path_checker>`__

-  ```features`` <#attribute-features>`__

-  ```failback`` <#attribute-failback>`__

-  ```prio`` <#attribute-prio>`__

-  ```prio_args`` <#attribute-prio_args>`__

-  ```no_path_retry`` <#attribute-no_path_retry>`__

-  ```rr_min_io`` <#attribute-rr_min_io>`__

-  ```rr_weight`` <#attribute-rr_weight>`__

-  ```fast_io_fail_tmo`` <#attribute-fast_io_fail_tmo>`__

-  ```dev_loss_tmo`` <#attribute-dev_loss_tmo>`__

-  ```flush_on_last_del`` <#attribute-flush_on_last_del>`__

    **Note**

    Whenever a hardware\_handler is specified, it is your responsibility
    to ensure that the appropriate kernel module is loaded to support
    the specified interface. These modules can be found in
    **``/lib/modules/`uname
            -r`/kernel/drivers/scsi/device_handler/ ``**. The requisite
    module should be integrated into the initrd to ensure the necessary
    discovery and failover-failback capacity is available during boot
    time. Example,

    ::

        # echo scsi_dh_alua >> /etc/initramfs-tools/modules  ## append module to file
        # update-initramfs -u -k all

The following example shows a device entry in the multipath
configuration file.

::

    #devices {
    #   device {
    #       vendor          "COMPAQ  "
    #       product         "MSA1000         "
    #       path_grouping_policy    multibus
    #       path_checker        tur
    #       rr_weight       priorities
    #   }
    #}

The spacing reserved in the **vendor**, **product**, and **revision**
fields are significant as multipath is performing a direct match against
these attributes, whose format is defined by the SCSI specification,
specifically the `Standard
INQUIRY <http://en.wikipedia.org/wiki/SCSI_Inquiry_Command>`__ command.
When quotes are used, the vendor, product, and revision fields will be
interpreted strictly according to the spec. Regular expressions may be
integrated into the quoted strings. Should a field be defined without
the requisite spacing, multipath will copy the string into the properly
sized buffer and pad with the appropriate number of spaces. The
specification expects the entire field to be populated by printable
characters or spaces, as seen in the example above

-  vendor: 8 characters

-  product: 16 characters

-  revision: 4 characters

To create a more robust configuration file, regular expressions can also
be used. Operators include **^ $ [ ] . \* ? +**. Examples of functional
regular expressions can be found by examining the live multipath
database and ``multipath.conf
      ``\ example files found in
``/usr/share/doc/multipath-tools/examples:``

::

    # echo 'show config' | multipathd -k

DMM Administration and Troubleshooting
======================================

Resizing an Online Multipath Device
-----------------------------------

If you need to resize an online multipath device, use the following
procedure

Resize your physical device. This is storage platform specific.

Use the following command to find the paths to the LUN:

::

    # multipath -l

Resize your paths. For SCSI devices, writing 1 to the ``rescan`` file
for the device causes the SCSI driver to rescan, as in the following
command:

::

    # echo 1 > /sys/block/device_name/device/rescan

Resize your multipath device by running the multipathd resize command:

::

    # multipathd -k 'resize map mpatha'

Resize the file system (assuming no LVM or DOS partitions are used):

::

    # resize2fs /dev/mapper/mpatha

Moving root File Systems from a Single Path Device to a Multipath Device
------------------------------------------------------------------------

This is dramatically simplified by the use of UUIDs to identify devices
as an intrinsic label. Simply install **multipath-tools-boot** and
reboot. This will rebuild the initial ramdisk and afford multipath the
opportunity to build it's paths before the root file system is mounted
by UUID.

    **Note**

    Whenever ``multipath.conf`` is updated, so should the initrd by
    executing ``update-initramfs -u -k
            all``. The reason being is ``multipath.conf`` is copied to
    the ramdisk and is integral to determining the available devices for
    grouping via it's blacklist and device sections.

Moving swap File Systems from a Single Path Device to a Multipath Device
------------------------------------------------------------------------

The procedure is exactly the same as illustrated in the previous section
called `Moving root File Systems from a Single Path to a Multipath
Device <#multipath-moving-rootfs-from-single-path-to-multipath-device>`__.

The Multipath Daemon
--------------------

If you find you have trouble implementing a multipath configuration, you
should ensure the multipath daemon is running as described in `"Setting
up DM-Multipath" <#multipath-setting-up-dm-multipath>`__. The
MULTIPATHD\_C daemon must be running in order to use multipathd devices.
Also see section `Troubleshooting with the multipathd interactive
console <#multipath-interacting-with-multipathd>`__ concerning
interacting with MULTIPATHD\_C as a debugging aid.

Issues with queue\_if\_no\_path
-------------------------------

If **features "1 queue\_if\_no\_path"** is specified in the
``/etc/multipath.conf`` file, then any process that uses I/O will hang
until one or more paths are restored. To avoid this, set the
**`no\_path\_retry <#attribute-no_path_retry>`__ N** parameter in the
``/etc/multipath.conf``.

When you set the **no\_path\_retry** parameter, remove the **features "1
queue\_if\_no\_path"** option from the ``/etc/multipath.conf`` file as
well. If, however, you are using a multipathed device for which the
``features "1
      queue_if_no_path"`` option is set as a compiled in default, as it
is for many SAN devices, you must add ``features "0"`` to override this
default. You can do this by copying the existing **devices** section,
and just that section (not the entire file), from
``/usr/share/doc/multipath-tools/examples/multipath.conf.annotated.gz``
into ``/etc/multipath.conf`` and editing to suit your needs.

If you need to use the ``features "1
      queue_if_no_path"`` option and you experience the issue noted
here, use the ``dmsetup`` command to edit the policy at runtime for a
particular LUN (that is, for which all the paths are unavailable). For
example, if you want to change the policy on the multipath device
``mpathc`` from ``"queue_if_no_path"`` to ``
      "fail_if_no_path"``, execute the following command.

::

    # dmsetup message mpathc 0 "fail_if_no_path"

    **Note**

    You must specify the ``mpathN`` alias rather than the path

Multipath Command Output
------------------------

When you create, modify, or list a multipath device, you get a printout
of the current device setup. The format is as follows. For each
multipath device:

::

       action_if_any: alias (wwid_if_different_from_alias) dm_device_name_if_known vendor,product
       size=size features='features' hwhandler='hardware_handler' wp=write_permission_if_known

For each path group:

::

      -+- policy='scheduling_policy' prio=prio_if_known
      status=path_group_status_if_known

For each path:

::

       `- host:channel:id:lun devnode major:minor dm_status_if_known path_status
      online_status

For example, the output of a multipath command might appear as follows:

::

      3600d0230000000000e13955cc3757800 dm-1 WINSYS,SF2372
      size=269G features='0' hwhandler='0' wp=rw
      |-+- policy='round-robin 0' prio=1 status=active
      | `- 6:0:0:0 sdb 8:16  active ready  running
      `-+- policy='round-robin 0' prio=1 status=enabled
        `- 7:0:0:0 sdf 8:80  active ready  running

If the path is up and ready for I/O, the status of the path is **ready**
or *ghost*. If the path is down, the status is **faulty** or **shaky**.
The path status is updated periodically by the MULTIPATHD\_C daemon
based on the polling interval defined in the ``/etc/multipath.conf``
file.

The dm status is similar to the path status, but from the kernel's point
of view. The dm status has two states: **failed**, which is analogous to
**faulty**, and **active** which covers all other path states.
Occasionally, the path state and the dm state of a device will
temporarily not agree.

The possible values for **online\_status** are **running** and
**offline**. A status of *offline* means that the SCSI device has been
disabled.

    **Note**

    When a multipath device is being created or modified , the path
    group status, the dm device name, the write permissions, and the dm
    status are not known. Also, the features are not always correct

Multipath Queries with multipath Command
----------------------------------------

You can use the **-l**\ and **-ll** options of the **multipath** command
to display the current multipath configuration. The **-l** option
displays multipath topology gathered from information in sysfs and the
device mapper. The **-ll** option displays the information the **-l**
displays in addition to all other available components of the system.

When displaying the multipath configuration, there are three verbosity
levels you can specify with the **-v** option of the multipath command.
Specifying **-v0** yields no output. Specifying\ **-v1** outputs the
created or updated multipath names only, which you can then feed to
other tools such as kpartx. Specifying **-v2** prints all detected
paths, multipaths, and device maps.

    **Note**

    The default **verbosity** level of multipath is **2** and can be
    globally modified by defining the `verbosity
    attribute <#attribute-verbosity>`__ in the **defaults** section of
    ``multipath.conf``.

The following example shows the output of a **multipath -l** command.

::

    # multipath -l
      3600d0230000000000e13955cc3757800 dm-1 WINSYS,SF2372
      size=269G features='0' hwhandler='0' wp=rw
      |-+- policy='round-robin 0' prio=1 status=active
      | `- 6:0:0:0 sdb 8:16  active ready  running
      `-+- policy='round-robin 0' prio=1 status=enabled
        `- 7:0:0:0 sdf 8:80  active ready  running

The following example shows the output of a **multipath -ll** command.

::

    # multipath -ll
      3600d0230000000000e13955cc3757801 dm-10 WINSYS,SF2372
      size=269G features='0' hwhandler='0' wp=rw
      |-+- policy='round-robin 0' prio=1 status=enabled
      | `- 19:0:0:1 sdc 8:32  active ready  running
      `-+- policy='round-robin 0' prio=1 status=enabled
        `- 18:0:0:1 sdh 8:112 active ready  running
        3600d0230000000000e13955cc3757803 dm-2 WINSYS,SF2372
        size=125G features='0' hwhandler='0' wp=rw
        `-+- policy='round-robin 0' prio=1 status=active
          |- 19:0:0:3 sde 8:64  active ready  running
            `- 18:0:0:3 sdj 8:144 active ready  running

Multipath Command Options
-------------------------

Table ? describes some options of the **multipath** command that you
might find useful.

+-----------------------+----------------------------------------------------+
| Option                | Description                                        |
+=======================+====================================================+
| **-l**                | Display the current multipath configuration        |
|                       | gathered from **sysfs** and the device mapper.     |
+-----------------------+----------------------------------------------------+
| **-ll**               | Display the current multipath configuration        |
|                       | gathered from **sysfs**, the device mapper, and    |
|                       | all other available components on the system.      |
+-----------------------+----------------------------------------------------+
| **-f device**         | Remove the named multipath device.                 |
+-----------------------+----------------------------------------------------+
| **-F**                | Remove all unused multipath devices.               |
+-----------------------+----------------------------------------------------+

Table: Useful multipath Command Options

Determining Device Mapper Entries with dmsetup Command
------------------------------------------------------

You can use the **dmsetup** command to find out which device mapper
entries match the **multipathed** devices.

The following command displays all the device mapper devices and their
major and minor numbers. The minor numbers determine the name of the dm
device. For example, a minor number of **3** corresponds to the
multipathed device **``/dev/dm-3``**.

::

    # dmsetup ls
    mpathd  (253, 4)
    mpathep1        (253, 12)
    mpathfp1        (253, 11)
    mpathb  (253, 3)
    mpathgp1        (253, 14)
    mpathhp1        (253, 13)
    mpatha  (253, 2)
    mpathh  (253, 9)
    mpathg  (253, 8)
    VolGroup00-LogVol01     (253, 1)
    mpathf  (253, 7)
    VolGroup00-LogVol00     (253, 0)
    mpathe  (253, 6)
    mpathbp1        (253, 10)
    mpathd  (253, 5)
      

Troubleshooting with the multipathd interactive console
-------------------------------------------------------

The **multipathd -k** command is an interactive interface to the
**multipathd** daemon. Entering this command brings up an interactive
multipath console. After entering this command, you can enter help to
get a list of available commands, you can enter a interactive command,
or you can enter **CTRL-D** to quit.

The multipathd interactive console can be used to troubleshoot problems
you may be having with your system. For example, the following command
sequence displays the multipath configuration, including the defaults,
before exiting the console. See the IBM article `"Tricks with
Multipathd" <http://www-01.ibm.com/support/docview.wss?uid=isg3T1011985>`__
for more examples.

::

    # multipathd -k
      > > show config
      > > CTRL-D

The following command sequence ensures that multipath has picked up any
changes to the multipath.conf,

::

    # multipathd -k
    > > reconfigure
    > > CTRL-D

Use the following command sequence to ensure that the path checker is
working properly.

::

    # multipathd -k
    > > show paths
    > > CTRL-D

Commands can also be streamed into multipathd using stdin like so:

::

    # echo 'show config' | multipathd -k

