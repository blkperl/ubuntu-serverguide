Package Management
==================

Ubuntu features a comprehensive package management system for
installing, upgrading, configuring, and removing software. In addition
to providing access to an organized base of over 35,000 software
packages for your Ubuntu computer, the package management facilities
also feature dependency resolution capabilities and software update
checking.

Several tools are available for interacting with Ubuntu's package
management system, from simple command-line utilities which may be
easily automated by system administrators, to a simple graphical
interface which is easy to use by those new to Ubuntu.

Introduction
============

Ubuntu's package management system is derived from the same system used
by the Debian GNU/Linux distribution. The package files contain all of
the necessary files, meta-data, and instructions to implement a
particular functionality or software application on your Ubuntu
computer.

Debian package files typically have the extension '.deb', and usually
exist in *repositories* which are collections of packages found on
various media, such as CD-ROM discs, or online. Packages are normally in
a pre-compiled binary format; thus installation is quick, and requires
no compiling of software.

Many complex packages use the concept of *dependencies*. Dependencies
are additional packages required by the principal package in order to
function properly. For example, the speech synthesis package festival
depends upon the package libasound2, which is a package supplying the
ALSA sound library needed for audio playback. In order for festival to
function, it and all of its dependencies must be installed. The software
management tools in Ubuntu will do this automatically.

dpkg
====

dpkg is a package manager for *Debian*-based systems. It can install,
remove, and build packages, but unlike other package management systems,
it cannot automatically download and install packages or their
dependencies. This section covers using dpkg to manage locally installed
packages:

-  To list all packages installed on the system, from a terminal prompt
   type:

   ::

-  Depending on the amount of packages on your system, this can generate
   a large amount of output. Pipe the output through grep to see if a
   specific package is installed:

   ::

   Replace *apache2* with any package name, part of a package name, or
   other regular expression.

-  To list the files installed by a package, in this case the ufw
   package, enter:

   ::

-  If you are not sure which package installed a file, dpkg -S may be
   able to tell you. For example:

   ::


   The output shows that the ``/etc/host.conf`` belongs to the
   base-files package.

       **Note**

       Many files are automatically generated during the package install
       process, and even though they are on the filesystem, ``dpkg -S``
       may not know which package they belong to.

-  You can install a local *.deb* file by entering:

   ::

   Change ``zip_3.0-4_i386.deb`` to the actual file name of the local
   .deb file you wish to install.

-  Uninstalling a package can be accomplished by:

   ::

       **Caution**

       Uninstalling packages using dpkg, in most cases, is *NOT*
       recommended. It is better to use a package manager that handles
       dependencies to ensure that the system is in a consistent state.
       For example using ``dpkg -r zip`` will remove the zip package,
       but any packages that depend on it will still be installed and
       may no longer function correctly.

For more dpkg options see the man page: ``man dpkg``.

Apt-Get
=======

The apt-get command is a powerful command-line tool, which works with
Ubuntu's *Advanced Packaging Tool* (APT) performing such functions as
installation of new software packages, upgrade of existing software
packages, updating of the package list index, and even upgrading the
entire Ubuntu system.

Being a simple command-line tool, apt-get has numerous advantages over
other package management tools available in Ubuntu for server
administrators. Some of these advantages include ease of use over simple
terminal connections (SSH), and the ability to be used in system
administration scripts, which can in turn be automated by the cron
scheduling utility.

Some examples of popular uses for the apt-get utility:

-  **Install a Package**: Installation of packages using the apt-get
   tool is quite simple. For example, to install the network scanner
   nmap, type the following:

   ::

-  **Remove a Package**: Removal of a package (or packages) is also
   straightforward. To remove the package installed in the previous
   example, type the following:

   ::

       **Tip**

       **Multiple Packages**: You may specify multiple packages to be
       installed or removed, separated by spaces.

   Also, adding the *--purge* option to ``apt-get remove`` will remove
   the package configuration files as well. This may or may not be the
   desired effect, so use with caution.

-  **Update the Package Index**: The APT package index is essentially a
   database of available packages from the repositories defined in the
   ``/etc/apt/sources.list`` file and in the ``/etc/apt/sources.list.d``
   directory. To update the local package index with the latest changes
   made in the repositories, type the following:

   ::

-  **Upgrade Packages**: Over time, updated versions of packages
   currently installed on your computer may become available from the
   package repositories (for example security updates). To upgrade your
   system, first update your package index as outlined above, and then
   type:

   ::

   For information on upgrading to a new Ubuntu release see ?.

Actions of the apt-get command, such as installation and removal of
packages, are logged in the /var/log/dpkg.log log file.

For further information about the use of APT, read the comprehensive
`Debian APT User Manual <&debian-apt;>`__ or type:

::

Aptitude
========

Launching Aptitude with no command-line options, will give you a
menu-driven, text-based front-end to the *Advanced Packaging Tool* (APT)
system. Many of the common package management functions, such as
installation, removal, and upgrade, can be performed in Aptitude with
single-key commands, which are typically lowercase letters.

Aptitude is best suited for use in a non-graphical terminal environment
to ensure proper functioning of the command keys. You may start the
menu-driven interface of Aptitude as a normal user by typing the
following command at a terminal prompt:

::

When Aptitude starts, you will see a menu bar at the top of the screen
and two panes below the menu bar. The top pane contains package
categories, such as *New Packages* and *Not Installed Packages*. The
bottom pane contains information related to the packages and package
categories.

Using Aptitude for package management is relatively straightforward, and
the user interface makes common tasks simple to perform. The following
are examples of common package management functions as performed in
Aptitude:

-  **Install Packages**: To install a package, locate the package via
   the *Not Installed Packages* package category, by using the keyboard
   arrow keys and the ENTER key. Highlight the desired package, then
   press the + key. The package entry should turn *green*, indicating
   that it has been marked for installation. Now press g to be presented
   with a summary of package actions. Press g again, and downloading and
   installation of the package will commence. When finished, press
   ENTER, to return to the menu.

-  **Remove Packages**: To remove a package, locate the package via the
   *Installed Packages* package category, by using the keyboard arrow
   keys and the ENTER key. Highlight the desired package you wish to
   remove, then press the - key. The package entry should turn *pink*,
   indicating it has been marked for removal. Now press g to be
   presented with a summary of package actions. Press g again, and
   removal of the package will commence. When finished, press ENTER, to
   return to the menu.

-  **Update Package Index**: To update the package index, simply press
   the u key. Updating of the package index will commence.

-  **Upgrade Packages**: To upgrade packages, perform the update of the
   package index as detailed above, and then press the U key to mark all
   packages with updates. Now press g whereby you'll be presented with a
   summary of package actions. Press g again, and the download and
   installation will commence. When finished, press ENTER, to return to
   the menu.

The first column of information displayed in the package list in the top
pane, when actually viewing packages lists the current state of the
package, and uses the following key to describe the state of the
package:

-  **i**: Installed package

-  **c**: Package not installed, but package configuration remains on
   system

-  **p**: Purged from system

-  **v**: Virtual package

-  **B**: Broken package

-  **u**: Unpacked files, but package not yet configured

-  **C**: Half-configured - Configuration failed and requires fix

-  **H**: Half-installed - Removal failed and requires fix

To exit Aptitude, simply press the q key and confirm you wish to exit.
Many other functions are available from the Aptitude menu by pressing
the F10 key.

Command Line Aptitude
---------------------

You can also use Aptitude as a command-line tool, similar to apt-get. To
install the nmap package with all necessary dependencies, as in the
apt-get example, you would use the following command:

::

To remove the same package, you would use the command:

::

Consult the man pages for more details of command line options for
Aptitude.

Automatic Updates
=================

The unattended-upgrades package can be used to automatically install
updated packages, and can be configured to update all packages or just
install security updates. First, install the package by entering the
following in a terminal:

::

To configure unattended-upgrades, edit
``/etc/apt/apt.conf.d/50unattended-upgrades`` and adjust the following
to fit your needs:

::

    Unattended-Upgrade::Allowed-Origins {
            "Ubuntu DISTRO-SHORT-CODENAME-security";
    //      "Ubuntu DISTRO-SHORT-CODENAME-updates";
    };

Certain packages can also be *blacklisted* and therefore will not be
automatically updated. To blacklist a package, add it to the list:

::

    Unattended-Upgrade::Package-Blacklist {
    //      "vim";
    //      "libc6";
    //      "libc6-dev";
    //      "libc6-i686";
    };

    **Note**

    The double *“//”* serve as comments, so whatever follows "//" will
    not be evaluated.

To enable automatic updates, edit ``/etc/apt/apt.conf.d/10periodic`` and
set the appropriate apt configuration options:

::

    APT::Periodic::Update-Package-Lists "1";
    APT::Periodic::Download-Upgradeable-Packages "1";
    APT::Periodic::AutocleanInterval "7";
    APT::Periodic::Unattended-Upgrade "1";

The above configuration updates the package list, downloads, and
installs available upgrades every day. The local download archive is
cleaned every week.

    **Note**

    You can read more about apt Periodic configuration options in the
    ``/etc/cron.daily/apt`` script header.

The results of unattended-upgrades will be logged to
``/var/log/unattended-upgrades``.

Notifications
-------------

Configuring *Unattended-Upgrade::Mail* in
``/etc/apt/apt.conf.d/50unattended-upgrades`` will enable
unattended-upgrades to email an administrator detailing any packages
that need upgrading or have problems.

Another useful package is apticron. apticron will configure a cron job
to email an administrator information about any packages on the system
that have updates available, as well as a summary of changes in each
package.

To install the apticron package, in a terminal enter:

::

Once the package is installed edit ``/etc/apticron/apticron.conf``, to
set the email address and other options:

::

    EMAIL="root@example.com"

Configuration
=============

Configuration of the *Advanced Packaging Tool* (APT) system repositories
is stored in the ``/etc/apt/sources.list`` file and the
``/etc/apt/sources.list.d`` directory. An example of this file is
referenced here, along with information on adding or removing repository
references from the file.

You may edit the file to enable repositories or disable them. For
example, to disable the requirement of inserting the Ubuntu CD-ROM
whenever package operations occur, simply comment out the appropriate
line for the CD-ROM, which appears at the top of the file:

::

    # no more prompting for CD-ROM please
    # deb cdrom:[DISTRO-APT-CD-NAME - Release i386 (20111013.1)]/ DISTRO-SHORT-CODENAME main restricted

Extra Repositories
------------------

In addition to the officially supported package repositories available
for Ubuntu, there exist additional community-maintained repositories
which add thousands more packages for potential installation. Two of the
most popular are the *Universe* and *Multiverse* repositories. These
repositories are not officially supported by Ubuntu, but because they
are maintained by the community they generally provide packages which
are safe for use with your Ubuntu computer.

    **Note**

    Packages in the *Multiverse* repository often have licensing issues
    that prevent them from being distributed with a free operating
    system, and they may be illegal in your locality.

    **Warning**

    Be advised that neither the *Universe* or *Multiverse* repositories
    contain officially supported packages. In particular, there may not
    be security updates for these packages.

Many other package sources are available, sometimes even offering only
one package, as in the case of package sources provided by the developer
of a single application. You should always be very careful and cautious
when using non-standard package sources, however. Research the source
and packages carefully before performing any installation, as some
package sources and their packages could render your system unstable or
non-functional in some respects.

By default, the *Universe* and *Multiverse* repositories are enabled but
if you would like to disable them edit ``/etc/apt/sources.list`` and
comment the following lines:

::

    deb http://archive.ubuntu.com/ubuntu DISTRO-SHORT-CODENAME universe multiverse
    deb-src http://archive.ubuntu.com/ubuntu DISTRO-SHORT-CODENAME universe multiverse

    deb http://us.archive.ubuntu.com/ubuntu/ DISTRO-SHORT-CODENAME universe
    deb-src http://us.archive.ubuntu.com/ubuntu/ DISTRO-SHORT-CODENAME universe
    deb http://us.archive.ubuntu.com/ubuntu/ DISTRO-SHORT-CODENAME-updates universe
    deb-src http://us.archive.ubuntu.com/ubuntu/ DISTRO-SHORT-CODENAME-updates universe

    deb http://us.archive.ubuntu.com/ubuntu/ DISTRO-SHORT-CODENAME multiverse
    deb-src http://us.archive.ubuntu.com/ubuntu/ DISTRO-SHORT-CODENAME multiverse
    deb http://us.archive.ubuntu.com/ubuntu/ DISTRO-SHORT-CODENAME-updates multiverse
    deb-src http://us.archive.ubuntu.com/ubuntu/ DISTRO-SHORT-CODENAME-updates multiverse

    deb http://security.ubuntu.com/ubuntu DISTRO-SHORT-CODENAME-security universe
    deb-src http://security.ubuntu.com/ubuntu DISTRO-SHORT-CODENAME-security universe
    deb http://security.ubuntu.com/ubuntu DISTRO-SHORT-CODENAME-security multiverse
    deb-src http://security.ubuntu.com/ubuntu DISTRO-SHORT-CODENAME-security multiverse

References
==========

Most of the material covered in this chapter is available in man pages,
many of which are available online.

-  The
   `InstallingSoftware <https://help.ubuntu.com/community/InstallingSoftware>`__
   Ubuntu wiki page has more information.

-  For more dpkg details see the `dpkg man
   page <http://manpages.ubuntu.com/manpages/&distro-short-codename;/en/man1/dpkg.1.html>`__.

-  The `APT HOWTO <http://www.debian.org/doc/manuals/apt-howto/>`__ and
   `apt-get man
   page <http://manpages.ubuntu.com/manpages/&distro-short-codename;/en/man8/apt-get.8.html>`__
   contain useful information regarding apt-get usage.

-  See the `aptitude man
   page <http://manpages.ubuntu.com/manpages/&distro-short-codename;/man8/aptitude.8.html>`__
   for more aptitude options.

-  The `Adding Repositories HOWTO (Ubuntu
   Wiki) <https://help.ubuntu.com/community/Repositories/Ubuntu>`__ page
   contains more details on adding repositories.


