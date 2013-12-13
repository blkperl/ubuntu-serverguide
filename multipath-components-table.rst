+---------------------+------------------------------------------------------+
| Component           | Description                                          |
+=====================+======================================================+
| **dm\_multipath     | Reroutes I/O and supports **failover** for paths and |
| kernel module**     | path groups.                                         |
+---------------------+------------------------------------------------------+
| **multipath         | Lists and configures **multipath** devices. Normally |
| command**           | started up with ``/etc/rc.sysinit``, it can also be  |
|                     | started up by a udev program whenever a block device |
|                     | is added or it can be run by the initramfs file      |
|                     | system.                                              |
+---------------------+------------------------------------------------------+
| **multipathd        | Monitors paths; as paths fail and come back, it may  |
| daemon**            | initiate path group switches. Provides for           |
|                     | interactive changes to **multipath** devices. This   |
|                     | daemon must be restarted for any changes to the      |
|                     | ``/etc/multipath.conf`` file to take effect.         |
+---------------------+------------------------------------------------------+
| **kpartx command**  | Creates device mapper devices for the partitions on  |
|                     | a device It is necessary to use this command for     |
|                     | DOS-based partitions with DM-Multipath. The kpartx   |
|                     | is provided in its own package, but the              |
|                     | **multipath-tools** package depends on it.           |
+---------------------+------------------------------------------------------+

Table: DM-Multipath Components

