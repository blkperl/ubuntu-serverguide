Clustering
==========

DRBD
====

Distributed Replicated Block Device (DRBD) mirrors block devices between
multiple hosts. The replication is transparent to other applications on
the host systems. Any block device hard disks, partitions, RAID devices,
logical volumes, etc can be mirrored.

To get started using drbd, first install the necessary packages. From a
terminal enter:

::

    **Note**

    If you are using the *virtual kernel* as part of a virtual machine
    you will need to manually compile the drbd module. It may be easier
    to install the linux-server package inside the virtual machine.

This section covers setting up a drbd to replicate a separate ``/srv``
partition, with an ext3 filesystem between two hosts. The partition size
is not particularly relevant, but both partitions need to be the same
size.

Configuration
-------------

The two hosts in this example will be called *drbd01* and *drbd02*. They
will need to have name resolution configured either through DNS or the
``/etc/hosts`` file. See ? for details.

-  To configure drbd, on the first host edit ``/etc/drbd.conf``:

   ::

       global { usage-count no; }
       common { syncer { rate 100M; } }
       resource r0 {
               protocol C;
               startup {
                       wfc-timeout  15;
                       degr-wfc-timeout 60;
               }
               net {
                       cram-hmac-alg sha1;
                       shared-secret "secret";
               }
               on drbd01 {
                       device /dev/drbd0;
                       disk /dev/sdb1;
                       address 192.168.0.1:7788;
                       meta-disk internal;
               }
               on drbd02 {
                       device /dev/drbd0;
                       disk /dev/sdb1;
                       address 192.168.0.2:7788;
                       meta-disk internal;
               }
       } 

       **Note**

       There are many other options in ``/etc/drbd.conf``, but for this
       example their default values are fine.

-  Now copy ``/etc/drbd.conf`` to the second host:

   ::

-  And, on *drbd02* move the file to ``/etc``:

   ::

-  Now using the drbdadm utility initialize the meta data storage. On
   each server execute:

   ::

-  Next, on both hosts, start the drbd daemon:

   ::

-  On the *drbd01*, or whichever host you wish to be the primary, enter
   the following:

   ::

-  After executing the above command, the data will start syncing with
   the secondary host. To watch the progress, on *drbd02* enter the
   following:

   ::

   To stop watching the output press *Ctrl+c*.

-  Finally, add a filesystem to ``/dev/drbd0`` and mount it:

   ::


Testing
-------

To test that the data is actually syncing between the hosts copy some
files on the *drbd01*, the primary, to ``/srv``:

::

Next, unmount ``/srv``:

::

*Demote* the *primary* server to the *secondary* role:

::

Now on the *secondary* server *promote* it to the *primary* role:

::

Lastly, mount the partition:

::

Using *ls* you should see ``/srv/default`` copied from the former
*primary* host *drbd01*.

References
----------

-  For more information on DRBD see the `DRBD web
   site <http://www.drbd.org/>`__.

-  The `drbd.conf man
   page <http://manpages.ubuntu.com/manpages/&distro-short-codename;/en/man5/drbd.conf.5.html>`__
   contains details on the options not covered in this guide.

-  Also, see the `drbdadm man
   page <http://manpages.ubuntu.com/manpages/&distro-short-codename;/en/man8/drbdadm.8.html>`__.

-  The `DRBD Ubuntu Wiki <https://help.ubuntu.com/community/DRBD>`__
   page also has more information.


