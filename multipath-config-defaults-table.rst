+--------------------------+--------------------------------------------------+
| Attribute                | Description                                      |
+==========================+==================================================+
| **polling\_interval**    | Specifies the interval between two path checks   |
|                          | in seconds. For properly functioning paths, the  |
|                          | interval between checks will gradually increase  |
|                          | to (4 \* **polling\_interval**). The default     |
|                          | value is **5**.                                  |
+--------------------------+--------------------------------------------------+
| **udev\_dir**            | The directory where udev device nodes are        |
|                          | created. The default value is /dev.              |
+--------------------------+--------------------------------------------------+
| **multipath\_dir**       | The directory where the dynamic shared objects   |
|                          | are stored. The default value is system          |
|                          | dependent, commonly ``/lib/multipath``.          |
+--------------------------+--------------------------------------------------+
| **verbosity**            | The default verbosity. Higher values increase    |
|                          | the verbosity level. Valid levels are between 0  |
|                          | and 6. The default value is 2.                   |
+--------------------------+--------------------------------------------------+
| **path\_selector**       | Specifies the default algorithm to use in        |
|                          | determining what path to use for the next I/O    |
|                          | operation. Possible values include:              |
|                          |                                                  |
|                          | -  **round-robin 0**: Loop through every path in |
|                          |    the path group, sending the same amount of    |
|                          |    I/O to each.                                  |
|                          |                                                  |
|                          | -  **queue-length 0**: Send the next bunch of    |
|                          |    I/O down the path with the least number of    |
|                          |    outstanding I/O requests.                     |
|                          |                                                  |
|                          | -  **service-time 0**: Send the next bunch of    |
|                          |    I/O down the path with the shortest estimated |
|                          |    service time, which is determined by dividing |
|                          |    the total size of the outstanding I/O to each |
|                          |    path by its relative throughput.              |
|                          |                                                  |
|                          | The default value is **round-robin 0**.          |
+--------------------------+--------------------------------------------------+
| **path\_grouping\_policy | Specifies the default path grouping policy to    |
| **                       | apply to unspecified multipaths. Possible values |
|                          | include:                                         |
|                          |                                                  |
|                          | -  **failover** = 1 path per priority group      |
|                          |                                                  |
|                          | -  **multibus** = all valid paths in 1 priority  |
|                          |    group                                         |
|                          |                                                  |
|                          | -  **group\_by\_serial** = 1 priority group per  |
|                          |    detected serial number                        |
|                          |                                                  |
|                          | -  **group\_by\_prio** = 1 priority group per    |
|                          |    path priority value                           |
|                          |                                                  |
|                          | -  **group\_by\_node\_name** = 1 priority group  |
|                          |    per target node name.                         |
|                          |                                                  |
|                          | The default value is **failover.**               |
+--------------------------+--------------------------------------------------+
| **getuid\_callout**      | Specifies the default program and arguments to   |
|                          | call out to obtain a unique path identifier. An  |
|                          | absolute path is required.                       |
|                          | The default value is **/lib/udev/scsi\_id        |
|                          | --whitelisted --device=/dev/%n.**                |
+--------------------------+--------------------------------------------------+
| **prio**                 | Specifies the default function to call to obtain |
|                          | a path priority value. For example, the ALUA     |
|                          | bits in SPC-3 provide an exploitable prio value. |
|                          | Possible values include:                         |
|                          |                                                  |
|                          | -  **const**: Set a priority of 1 to all paths.  |
|                          |                                                  |
|                          | -  **emc**: Generate the path priority for EMC   |
|                          |    arrays.                                       |
|                          |                                                  |
|                          | -  **alua**: Generate the path priority based on |
|                          |    the SCSI-3 ALUA settings.                     |
|                          |                                                  |
|                          | -  **netapp**: Generate the path priority for    |
|                          |    NetApp arrays.                                |
|                          |                                                  |
|                          | -  **rdac**: Generate the path priority for      |
|                          |    LSI/Engenio RDAC controller.                  |
|                          |                                                  |
|                          | -  **hp\_sw**: Generate the path priority for    |
|                          |    Compaq/HP controller in active/standby mode.  |
|                          |                                                  |
|                          | -  **hds**: Generate the path priority for       |
|                          |    Hitachi HDS Modular storage arrays.           |
|                          |                                                  |
|                          | The default value is **const**.                  |
+--------------------------+--------------------------------------------------+
| **prio\_args**           | The arguments string passed to the prio function |
|                          | Most prio functions do not need arguments. The   |
|                          | datacore prioritizer need one. Example,          |
|                          | **"timeout=1000 preferredsds=foo"**. The default |
|                          | value is (null) **""**.                          |
+--------------------------+--------------------------------------------------+
| **features**             | The extra features of multipath devices. The     |
|                          | only existing feature is                         |
|                          | **queue\_if\_no\_path**, which is the same as    |
|                          | setting **no\_path\_retry** to **queue**. For    |
|                          | information on issues that may arise when using  |
|                          | this feature, see Section, `"Issues with         |
|                          | queue\_if\_no\_path                              |
|                          | feature" <#multipath-issues-with-queue_if_no_pat |
|                          | h>`__.                                           |
+--------------------------+--------------------------------------------------+
| **path\_checker**        | Specifies the default method used to determine   |
|                          | the state of the paths. Possible values include: |
|                          |                                                  |
|                          | -  **readsector0**: Read the first sector of the |
|                          |    device.                                       |
|                          |                                                  |
|                          | -  **tur**: Issue a TEST UNIT READY to the       |
|                          |    device.                                       |
|                          |                                                  |
|                          | -  **emc\_clariion**: Query the EMC Clariion     |
|                          |    specific EVPD page 0xC0 to determine the      |
|                          |    path.                                         |
|                          |                                                  |
|                          | -  **hp\_sw**: Check the path state for HP       |
|                          |    storage arrays with Active/Standby firmware.  |
|                          |                                                  |
|                          | -  **rdac**: Check the path status for           |
|                          |    LSI/Engenio RDAC storage controller.          |
|                          |                                                  |
|                          | -  **directio**: Read the first sector with      |
|                          |    direct I/O.                                   |
|                          |                                                  |
|                          | The default value is **directio**.               |
+--------------------------+--------------------------------------------------+
| **failback**             | Manages path group failback.                     |
|                          |                                                  |
|                          | -  A value of **immediate** specifies immediate  |
|                          |    failback to the highest priority path group   |
|                          |    that contains active paths.                   |
|                          |                                                  |
|                          | -  A value of **manual** specifies that there    |
|                          |    should not be immediate failback but that     |
|                          |    failback can happen only with operator        |
|                          |    intervention.                                 |
|                          |                                                  |
|                          | -  A numeric value greater than zero specifies   |
|                          |    deferred failback, expressed in seconds.      |
|                          |                                                  |
|                          | The default value is **manual**.                 |
+--------------------------+--------------------------------------------------+
| **rr\_min\_io**          | Specifies the number of I/O requests to route to |
|                          | a path before switching to the next path in the  |
|                          | current path group.                              |
|                          | The default value is ``1000``.                   |
+--------------------------+--------------------------------------------------+
| **rr\_weight**           | If set to **priorities**, then instead of        |
|                          | sending **rr\_min\_io** requests to a path       |
|                          | before calling **path\_selector** to choose the  |
|                          | next path, the number of requests to send is     |
|                          | determined by **rr\_min\_io** times the path's   |
|                          | priority, as determined by the prio function. If |
|                          | set to **uniform**, all path weights are equal.  |
|                          | The default value is **uniform**.                |
+--------------------------+--------------------------------------------------+
| **no\_path\_retry**      | A numeric value for this attribute specifies the |
|                          | number of times the system should attempt to use |
|                          | a failed path before disabling queueing. A value |
|                          | of fail indicates **immediate** failure, without |
|                          | queueing. A value of **queue** indicates that    |
|                          | queueing should not stop until the path is       |
|                          | fixed.                                           |
|                          | The default value is ``0``.                      |
+--------------------------+--------------------------------------------------+
| **user\_friendly\_names* | If set to yes, specifies that the system should  |
| *                        | use the ``/etc/multipath/bindings`` file to      |
|                          | assign a persistent and unique **alias** to the  |
|                          | **multipath**, in the form of mpathn. If set to  |
|                          | no, specifies that the system should use the     |
|                          | WWID as the **alias** for the **multipath**. In  |
|                          | either case, what is specified here will be      |
|                          | overridden by any device-specific aliases you    |
|                          | specify in the multipaths section of the         |
|                          | configuration file.                              |
|                          | The default value is **no**.                     |
+--------------------------+--------------------------------------------------+
| **queue\_without\_daemon | If set to no, the **multipathd** daemon will     |
| **                       | disable queueing for all devices when it is shut |
|                          | down.                                            |
|                          | The default value is **yes**.                    |
+--------------------------+--------------------------------------------------+
| **flush\_on\_last\_del** | If set to yes, then **multipath** will disable   |
|                          | queueing when the last path to a device has been |
|                          | deleted.                                         |
|                          | The default value is **no**.                     |
+--------------------------+--------------------------------------------------+
| **max\_fds**             | Sets the maximum number of open file descriptors |
|                          | that can be opened by **multipath** and the      |
|                          | **multipathd** daemon. This is equivalent to the |
|                          | ulimit -n command. A value of max will set this  |
|                          | to the system limit from                         |
|                          | ``/proc/sys/fs/nr_open``. If this is not set,    |
|                          | the maximum number of open file descriptors is   |
|                          | taken from the calling process; it is usually    |
|                          | 1024. To be safe, this should be set to the      |
|                          | maximum number of paths plus 32, if that number  |
|                          | is greater than 1024.                            |
+--------------------------+--------------------------------------------------+
| **checker\_timer**       | The timeout to use for path checkers that issue  |
|                          | SCSI commands with an explicit timeout, in       |
|                          | seconds.                                         |
|                          | The default value is taken from                  |
|                          | ``/sys/block/sdx/device/timeout``, which is      |
|                          | ``30`` seconds as of 12.04 LTS                   |
+--------------------------+--------------------------------------------------+
| **fast\_io\_fail\_tmo**  | The number of seconds the SCSI layer will wait   |
|                          | after a problem has been detected on an FC       |
|                          | remote port before failing I/O to devices on     |
|                          | that remote port. This value should be smaller   |
|                          | than the value of dev\_loss\_tmo. Setting this   |
|                          | to off will disable the timeout.                 |
|                          | The default value is determined by the OS.       |
+--------------------------+--------------------------------------------------+
| **dev\_loss\_tmo**       | The number of seconds the SCSI layer will wait   |
|                          | after a problem has been detected on an FC       |
|                          | remote port before removing it from the system.  |
|                          | Setting this to infinity will set this to        |
|                          | 2147483647 seconds, or 68 years. The default     |
|                          | value is determined by the OS.                   |
+--------------------------+--------------------------------------------------+

Table: Multipath Configuration Defaults

