Other Useful Applications
=========================

There are many very useful applications developed by the Ubuntu Server
Team, and others that are well integrated with Ubuntu Server Edition,
that might not be well known. This chapter will showcase some useful
applications that can make administering an Ubuntu server, or many
Ubuntu servers, that much easier.

pam\_motd
=========

When logging into an Ubuntu server you may have noticed the informative
Message Of The Day (MOTD). This information is obtained and displayed
using a couple of packages:

-  *landscape-common:* provides the core libraries of landscape-client,
   which can be used to manage systems using the web based *Landscape*
   application. The package includes the /usr/bin/landscape-sysinfo
   utility which is used to gather the information displayed in the
   MOTD.

-  *update-notifier-common:* is used to automatically update the MOTD
   via pam\_motd module.

pam\_motd executes the scripts in ``/etc/update-motd.d`` in order based
on the number prepended to the script. The output of the scripts is
written to ``/var/run/motd``, keeping the numerical order, then
concatenated with ``/etc/motd.tail``.

You can add your own dynamic information to the MOTD. For example, to
add local weather information:

-  First, install the weather-util package:

   ::

-  The weather utility uses METAR data from the National Oceanic and
   Atmospheric Administration and forecasts from the National Weather
   Service. In order to find local information you will need the
   4-character ICAO location indicator. This can be determined by
   browsing to the `National Weather
   Service <http://www.weather.gov/tg/siteloc.shtml>`__ site.

   Although the National Weather Service is a United States government
   agency there are weather stations available world wide. However,
   local weather information for all locations outside the U.S. may not
   be available.

-  Create ``/usr/local/bin/local-weather``, a simple shell script to use
   weather with your local ICAO indicator:

   ::

       #!/bin/sh
       #
       #
       # Prints the local weather information for the MOTD.
       #
       #

       # Replace KINT with your local weather station.
       # Local stations can be found here: http://www.weather.gov/tg/siteloc.shtml

       echo
       weather -i KINT
       echo

-  Make the script executable:

   ::

-  Next, create a symlink to ``/etc/update-motd.d/98-local-weather``:

   ::

-  Finally, exit the server and re-login to view the new MOTD.

You should now be greeted with some useful information, and some
information about the local weather that may not be quite so useful.
Hopefully the local-weather example demonstrates the flexibility of
pam\_motd.

etckeeper
=========

etckeeper allows the contents of ``/etc`` be easily stored in Version
Control System (VCS) repository. It hooks into apt to automatically
commit changes to ``/etc`` when packages are installed or upgraded.
Placing ``/etc`` under version control is considered an industry best
practice, and the goal of etckeeper is to make this process as painless
as possible.

Install etckeeper by entering the following in a terminal:

::

The main configuration file, ``/etc/etckeeper/etckeeper.conf``, is
fairly simple. The main option is which VCS to use. By default etckeeper
is configured to use bzr for version control. The repository is
automatically initialized (and committed for the first time) during
package installation. It is possible to undo this by entering the
following command:

::

By default, etckeeper will commit uncommitted changes made to /etc
daily. This can be disabled using the AVOID\_DAILY\_AUTOCOMMITS
configuration option. It will also automatically commit changes before
and after package installation. For a more precise tracking of changes,
it is recommended to commit your changes manually, together with a
commit message, using:

::

Using the VCS commands you can view log information about files in
``/etc``:

::

To demonstrate the integration with the package management system,
install postfix:

::

When the installation is finished, all the postfix configuration files
should be committed to the repository:

::

For an example of how etckeeper tracks manual changes, add new a host to
``/etc/hosts``. Using bzr you can see which files have been modified:

::


Now commit the changes:

::

For more information on bzr see ?.

Byobu
=====

One of the most useful applications for any system administrator is
screen. It allows the execution of multiple shells in one terminal. To
make some of the advanced screen features more user friendly, and
provide some useful information about the system, the byobu package was
created.

When executing byobu pressing the *F9* key will bring up the
Configuration menu. This menu will allow you to:

-  View the Help menu

-  Change Byobu's background color

-  Change Byobu's foreground color

-  Toggle status notifications

-  Change the key binding set

-  Change the escape sequence

-  Create new windows

-  Manage the default windows

-  Byobu currently does not launch at login (toggle on)

The *key bindings* determine such things as the escape sequence, new
window, change window, etc. There are two key binding sets to choose
from *f-keys* and *screen-escape-keys*. If you wish to use the original
key bindings choose the *none* set.

byobu provides a menu which displays the Ubuntu release, processor
information, memory information, and the time and date. The effect is
similar to a desktop menu.

Using the *"Byobu currently does not launch at login (toggle on)"*
option will cause byobu to be executed any time a terminal is opened.
Changes made to byobu are on a per user basis, and will not affect other
users on the system.

One difference when using byobu is the *scrollback* mode. Press the *F7*
key to enter scrollback mode. Scrollback mode allows you to navigate
past output using *vi* like commands. Here is a quick list of movement
commands:

-  *h* - Move the cursor left by one character

-  *j* - Move the cursor down by one line

-  *k* - Move the cursor up by one line

-  *l* - Move the cursor right by one character

-  *0* - Move to the beginning of the current line

-  *$* - Move to the end of the current line

-  *G* - Moves to the specified line (defaults to the end of the buffer)

-  */* - Search forward

-  *?* - Search backward

-  *n* - Moves to the next match, either forward or backward

References
==========

-  See the `update-motd man
   page <http://manpages.ubuntu.com/manpages/&distro-short-codename;/en/man5/update-motd.5.html>`__
   for more options available to update-motd.

-  The Debian Package of the Day
   `weather <http://debaday.debian.net/2007/10/04/weather-check-weather-conditions-and-forecasts-on-the-command-line/>`__
   article has more details about using the weatherutility.

-  See the `etckeeper <http://kitenet.net/~joey/code/etckeeper/>`__ site
   for more details on using etckeeper.

-  The `etckeeper Ubuntu
   Wiki <https://help.ubuntu.com/community/etckeeper>`__ page.

-  For the latest news and information about bzr see the
   `bzr <http://bazaar-vcs.org/>`__ web site.

-  For more information on screen see the `screen web
   site <http://www.gnu.org/software/screen/>`__.

-  And the `Ubuntu Wiki
   screen <https://help.ubuntu.com/community/Screen>`__ page.

-  Also, see the byobu `project page <https://launchpad.net/byobu>`__
   for more information.


