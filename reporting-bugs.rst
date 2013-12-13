Appendix
========

Reporting Bugs in Ubuntu Server Edition
=======================================

While the Ubuntu Project attempts to release software with as few bugs
as possible, they do occur. You can help fix these bugs by reporting
ones that you find to the project. The Ubuntu Project uses
`Launchpad <https://launchpad.net/>`__ to track its bug reports. In
order to file a bug about Ubuntu Server on Launchpad, you will need to
`create an
account <https://help.launchpad.net/YourAccount/NewAccount>`__.

Reporting Bugs With ubuntu-bug
------------------------------

The preferred way to report a bug is with the ubuntu-bug command. The
ubuntu-bug tool gathers information about the system useful to
developers in diagnosing the reported problem that will then be included
in the bug report filed on Launchpad. Bug reports in Ubuntu need to be
filed against a specific software package, thus the name of the package
that the bug occurs in needs to be given to ubuntu-bug:

::

For example, to file a bug against the openssh-server package, you would
do:

::

You can specify either a binary package or the source package for
ubuntu-bug. Again using openssh-server as an example, you could also
generate the report against the source package for openssh-server,
openssh:

::

    **Note**

    See ? for more information about packages in Ubuntu.

The ubuntu-bug command will gather information about the system in
question, possibly including information specific to the specified
package, and then ask you what you would like to do with collected
information:

::



    *** Collecting problem information

    The collected information can be sent to the developers to improve the
    application. This might take a few minutes.
    ..........

    *** Send problem report to the developers?

    After the problem report has been sent, please fill out the form in the
    automatically opened web browser.

    What would you like to do? Your options are:
      S: Send report (1.7 KiB)
      V: View report
      K: Keep report file for sending later or copying to somewhere else
      C: Cancel
    Please choose (S/V/K/C):

The options available are:

-  **Send Report** Selecting Send Report submits the collected
   information to Launchpad as part of the process of filing a bug
   report. You will be given the opportunity to describe the situation
   that led up to the occurrence of the bug.

   ::

       *** Uploading problem information

       The collected information is being sent to the bug tracking system.
       This might take a few minutes.
       91%

       *** To continue, you must visit the following URL:

       https://bugs.launchpad.net/ubuntu/+source/postgresql-8.4/+filebug/kc6eSnTLnLxF8u0t3e56EukFeqJ?

       You can launch a browser now, or copy this URL into a browser on another
       computer.

       Choices:
         1: Launch a browser now
         C: Cancel
       Please choose (1/C):

   If you choose to start a browser, by default the text based web
   browser w3m will be used to finish filing the bug report.
   Alternately, you can copy the given URL to a currently running web
   browser.

-  **View Report** Selecting View Report causes the collected
   information to be displayed to the terminal for review.

   ::

       Package: postgresql 8.4.2-2
       PackageArchitecture: all
       Tags: lucid
       ProblemType: Bug
       ProcEnviron:
         LANG=en_US.UTF-8
         SHELL=/bin/bash
       Uname: Linux 2.6.32-16-server x86_64
       Dependencies:
         adduser 3.112ubuntu1
         base-files 5.0.0ubuntu10
         base-passwd 3.5.22
         coreutils 7.4-2ubuntu2
       ...

   After viewing the report, you will be brought back to the same menu
   asking what you would like to do with the report.

-  **Keep Report File** Selecting Keep Report File causes the gathered
   information to be written to a file. This file can then be used to
   later file a bug report or transferred to a different Ubuntu system
   for reporting. To submit the report file, simply give it as an
   argument to the ubuntu-bug command:

   ::

       What would you like to do? Your options are:
         S: Send report (1.7 KiB)
         V: View report
         K: Keep report file for sending later or copying to somewhere else
         C: Cancel
       Please choose (S/V/K/C): 
       Problem report file: /tmp/apport.postgresql.v4MQas.apport



       *** Send problem report to the developers?
       ...

-  **Cancel** Selecting Cancel causes the collected information to be
   discarded.

Reporting Application Crashes
-----------------------------

The software package that provides the ubuntu-bug utility, apport, can
be configured to trigger when applications crash. This is disabled by
default, as capturing a crash can be resource intensive depending on how
much memory the application that crashed was using as apport captures
and processes the core dump.

Configuring apport to capture information about crashing applications
requires a couple of steps. First, gdb needs to be installed; it is not
installed by default in Ubuntu Server Edition.

::

See ? for more information about managing packages in Ubuntu.

Once you have ensured that gdb is installed, open the file
``/etc/default/apport`` in your text editor, and change the *enabled*
setting to be **1** like so:

::

    # set this to 0 to disable apport, or to 1 to enable it
    # you can temporarily override this with
    # sudo service apport start force_start=1
    enabled=

    # set maximum core dump file size (default: 209715200 bytes == 200 MB)
    maxsize=209715200

Once you have completed editing ``/etc/default/apport``, start the
apport service:

::

After an application crashes, use the apport-cli command to search for
the existing saved crash report information:

::



    *** dash closed unexpectedly on 2010-03-11 at 21:40:59.

    If you were not doing anything confidential (entering passwords or other
    private information), you can help to improve the application by
    reporting
    the problem.

    What would you like to do? Your options are:
      R: Report Problem...
      I: Cancel and ignore future crashes of this program version
      C: Cancel
    Please choose (R/I/C):

Selecting *Report Problem* will walk you through similar steps as when
using ubuntu-bug. One important difference is that a crash report will
be marked as private when filed on Launchpad, meaning that it will be
visible to only a limited set of bug triagers. These triagers will
review the gathered data for private information before making the bug
report publicly visible.

Resources
---------

-  See the `Reporting
   Bugs <https://help.ubuntu.com/community/ReportingBugs>`__ Ubuntu wiki
   page.

-  Also, the `Apport <https://wiki.ubuntu.com/Apport>`__ page has some
   useful information. Though some of it pertains to using a GUI.


