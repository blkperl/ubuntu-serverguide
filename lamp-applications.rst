LAMP Applications
=================

Overview
========

LAMP installations (Linux + Apache + MySQL + PHP/Perl/Python) are a
popular setup for Ubuntu servers. There is a plethora of Open Source
applications written using the LAMP application stack. Some popular LAMP
applications are Wiki's, Content Management Systems, and Management
Software such as phpMyAdmin.

One advantage of LAMP is the substantial flexibility for different
database, web server, and scripting languages. Popular substitutes for
MySQL include PostgreSQL and SQLite. Python, Perl, and Ruby are also
frequently used instead of PHP. While Nginx, Cherokee and Lighttpd can
replace Apache.

The fastest way to get started is to install LAMP using tasksel. Tasksel
is a Debian/Ubuntu tool that installs multiple related packages as a
co-ordinated "task" onto your system. To install a LAMP server:

At a terminal prompt enter the following command:

::

After installing it you'll be able to install most *LAMP* applications
in this way:

-  Download an archive containing the application source files.

-  Unpack the archive, usually in a directory accessible to a web
   server.

-  Depending on where the source was extracted, configure a web server
   to serve the files.

-  Configure the application to connect to the database.

-  Run a script, or browse to a page of the application, to install the
   database needed by the application.

-  Once the steps above, or similar steps, are completed you are ready
   to begin using the application.

A disadvantage of using this approach is that the application files are
not placed in the file system in a standard way, which can cause
confusion as to where the application is installed. Another larger
disadvantage is updating the application. When a new version is
released, the same process used to install the application is needed to
apply updates.

Fortunately, a number of *LAMP* applications are already packaged for
Ubuntu, and are available for installation in the same way as non-LAMP
applications. Depending on the application some extra configuration and
setup steps may be needed, however.

This section covers how to install some *LAMP* applications.

Moin Moin
=========

MoinMoin is a Wiki engine implemented in Python, based on the PikiPiki
Wiki engine, and licensed under the GNU GPL.

Installation
------------

To install MoinMoin, run the following command in the command prompt:

::

You should also install apache2 web server. For installing apache2 web
server, please refer to ? sub-section in ? section.

Configuration
-------------

For configuring your first Wiki application, please run the following
set of commands. Let us assume that you are creating a Wiki named
*mywiki*:

::








Now you should configure MoinMoin to find your new Wiki *mywiki*. To
configure MoinMoin, open ``/etc/moin/mywiki.py`` file and change the
following line:

::

    data_dir = '/org/mywiki/data'

to

::

    data_dir = '/usr/share/moin/mywiki/data'

Also, below the *data\_dir* option add the *data\_underlay\_dir*:

::

    data_underlay_dir='/usr/share/moin/mywiki/underlay'

    **Note**

    If the ``/etc/moin/mywiki.py`` file does not exists, you should copy
    ``/usr/share/moin/config/wikifarm/mywiki.py`` file to
    ``/etc/moin/mywiki.py`` file and do the above mentioned change.

    **Note**

    If you have named your Wiki as *my\_wiki\_name* you should insert a
    line “("my\_wiki\_name", r".\*")” in ``/etc/moin/farmconfig.py``
    file after the line “("mywiki", r".\*")”.

Once you have configured MoinMoin to find your first Wiki application
*mywiki*, you should configure apache2 and make it ready for your Wiki
application.

You should add the following lines in
``/etc/apache2/sites-available/default`` file inside the “<VirtualHost
\*>” tag:

::

    ### moin
      ScriptAlias /mywiki "/usr/share/moin/mywiki/moin.cgi"
      alias /moin_static193 "/usr/share/moin/htdocs"
      <Directory /usr/share/moin/htdocs>
      Order allow,deny
      allow from all
      </Directory>
    ### end moin

Once you configure the apache2 web server and make it ready for your
Wiki application, you should restart it. You can run the following
command to restart the apache2 web server:

::

Verification
------------

You can verify the Wiki application and see if it works by pointing your
web browser to the following URL:

::

    http://localhost/mywiki

For more details, please refer to the `MoinMoin <http://moinmo.in/>`__
web site.

References
----------

-  For more information see the `moinmoin Wiki <http://moinmo.in/>`__.

-  Also, see the `Ubuntu Wiki
   MoinMoin <https://help.ubuntu.com/community/MoinMoin>`__ page.

MediaWiki
=========

MediaWiki is an web based Wiki software written in the PHP language. It
can either use MySQL or PostgreSQL Database Management System.

Installation
------------

Before installing MediaWiki you should also install Apache2, the PHP5
scripting language and Database a Management System. MySQL or PostgreSQL
are the most common, choose one depending on your need. Please refer to
those sections in this manual for installation instructions.

To install MediaWiki, run the following command in the command prompt:

::

For additional MediaWiki functionality see the mediawiki-extensions
package.

Configuration
-------------

The Apache configuration file ``mediawiki.conf`` for MediaWiki is
installed in ``/etc/apache2/conf.d/`` directory. You should uncomment
the following line in this file to access MediaWiki application.

::

    # Alias /mediawiki /var/lib/mediawiki

After you uncomment the above line, restart Apache server and access
MediaWiki using the following url:

::

    http://localhost/mediawiki/config/index.php

    **Tip**

    Please read the “Checking environment...” section in this page. You
    should be able to fix many issues by carefully reading this section.

Once the configuration is complete, you should copy the
``LocalSettings.php`` file to ``/etc/mediawiki`` directory:

::

You may also want to edit ``/etc/mediawiki/LocalSettings.php`` in order
to set the memory limit (disabled by default):

::

    ini_set( 'memory_limit', '64M' );

Extensions
----------

The extensions add new features and enhancements for the MediaWiki
application. The extensions give wiki administrators and end users the
ability to customize MediaWiki to their requirements.

You can download MediaWiki extensions as an archive file or checkout
from the Subversion repository. You should copy it to
``/var/lib/mediawiki/extensions`` directory. You should also add the
following line at the end of file: ``/etc/mediawiki/LocalSettings.php``.

::

    require_once "$IP/extensions/ExtentionName/ExtentionName.php";

References
----------

-  For more details, please refer to the
   `MediaWiki <http://www.mediawiki.org>`__ web site.

-  The `MediaWiki Administrators' Tutorial
   Guide <http://www.packtpub.com/Mediawiki/book>`__ contains a wealth
   of information for new MediaWiki administrators.

-  Also, the `Ubuntu Wiki
   MediaWiki <https://help.ubuntu.com/community/MediaWiki>`__ page is a
   good resource.

phpMyAdmin
==========

phpMyAdmin is a LAMP application specifically written for administering
MySQL servers. Written in PHP, and accessed through a web browser,
phpMyAdmin provides a graphical interface for database administration
tasks.

Installation
------------

Before installing phpMyAdmin you will need access to a MySQL database
either on the same host as that phpMyAdmin is installed on, or on a host
accessible over the network. For more information see ?. From a terminal
prompt enter:

::

At the prompt choose which web server to be configured for phpMyAdmin.
The rest of this section will use Apache2 for the web server.

In a browser go to *http://servername/phpmyadmin*, replacing
*serveranme* with the server's actual hostname. At the login, page enter
*root* for the *username*, or another MySQL user if you any setup, and
enter the MySQL user's password.

Once logged in you can reset the *root* password if needed, create
users, create/destroy databases and tables, etc.

Configuration
-------------

The configuration files for phpMyAdmin are located in
``/etc/phpmyadmin``. The main configuration file is
``/etc/phpmyadmin/config.inc.php``. This file contains configuration
options that apply globally to phpMyAdmin.

To use phpMyAdmin to administer a MySQL database hosted on another
server, adjust the following in ``/etc/phpmyadmin/config.inc.php``:

::

    $cfg['Servers'][$i]['host'] = 'db_server';

    **Note**

    Replace *db\_server* with the actual remote database server name or
    IP address. Also, be sure that the phpMyAdmin host has permissions
    to access the remote database.

Once configured, log out of phpMyAdmin and back in, and you should be
accessing the new server.

The ``config.header.inc.php`` and ``config.footer.inc.php`` files are
used to add a HTML header and footer to phpMyAdmin.

Another important configuration file is ``/etc/phpmyadmin/apache.conf``,
this file is symlinked to ``/etc/apache2/conf.d/phpmyadmin.conf``, and
is used to configure Apache2 to serve the phpMyAdmin site. The file
contains directives for loading PHP, directory permissions, etc. For
more information on configuring Apache2 see ?.

References
----------

-  The phpMyAdmin documentation comes installed with the package and can
   be accessed from the *phpMyAdmin Documentation* link (a question mark
   with a box around it) under the phpMyAdmin logo. The official docs
   can also be access on the
   `phpMyAdmin <http://www.phpmyadmin.net/home_page/docs.php>`__ site.

-  Also, `Mastering
   phpMyAdmin <http://www.packtpub.com/phpmyadmin-3rd-edition/book>`__
   is a great resource.

-  A third resource is the `phpMyAdmin Ubuntu
   Wiki <https://help.ubuntu.com/community/phpMyAdmin>`__ page.

WordPress
=========

Wordpress is a blog tool, publishing platform and CMS implemented in PHP
and licensed under the GNU GPLv2.

Installation
------------

To install WordPress, run the following comand in the command prompt:

::

        

You should also install apache2 web server and mysql server. For
installing apache2 web server, please refer to ? sub-section in ?
section. For installing mysql server, please refer to ? sub-section in ?
section.

Configuration
-------------

For configuring your first WordPress application, configure an apache
site. Open ``/etc/apache2/sites-available/wordpress`` and write the
following lines:

::

            Alias /blog /usr/share/wordpress
            Alias /blog/wp-content /var/lib/wordpress/wp-content
            <Directory /usr/share/wordpress>
                Options FollowSymLinks
                AllowOverride Limit Options FileInfo
                DirectoryIndex index.php
                Order allow,deny
                Allow from all
            </Directory>
            <Directory /var/lib/wordpress/wp-content>
                Options FollowSymLinks
                Order allow,deny
                Allow from all
            </Directory>
               

Enable this new WordPress site

::

        

Once you configure the apache2 web server and make it ready for your
WordPress application, you should restart it. You can run the following
command to restart the apache2 web server:

::

To facilitate multiple WordPress installations, the name of this
configuration file is based on the Host header of the HTTP request. This
means that you can have a configuration per VirtualHost by simply
matching the hostname portion of this configuration with your Apache
Virtual Host. e.g. /etc/wordpress/config-10.211.55.50.php,
/etc/wordpress/config-hostalias1.php, etc. These instructions assume you
can access Apache via the localhost hostname (perhaps by using an ssh
tunnel) if not, replace /etc/wordpress/config-localhost.php with
/etc/wordpress/config-NAME\_OF\_YOUR\_VIRTUAL\_HOST.php.

Once the configuration file is written, it is up to you to choose a
convention for username and password to mysql for each WordPress
database instance. This documentation shows only one, localhost,
example.

Now configure WordPress to use a mysql database. Open
``/etc/wordpress/config-localhost.php`` file and write the following
lines:

::

    <?php
    define('DB_NAME', 'wordpress');
    define('DB_USER', 'wordpress');
    define('DB_PASSWORD', 'yourpasswordhere');
    define('DB_HOST', 'localhost');
    define('WP_CONTENT_DIR', '/var/lib/wordpress/wp-content');
    ?>

Now create this mysql database. Open a temporary file with mysql
commands ``wordpress.sql`` and write the following lines:

::

    CREATE DATABASE wordpress;
    GRANT SELECT,INSERT,UPDATE,DELETE,CREATE,DROP,ALTER
    ON wordpress.*
    TO wordpress@localhost
    IDENTIFIED BY 'yourpasswordhere';
    FLUSH PRIVILEGES;

Execute these commands.

::

Your new WordPress can now be configured by visiting
http://localhost/blog/wp-admin/install.php. (Or
http://NAME_OF_YOUR_VIRTUAL_HOST/blog/wp-admin/install.php if your
server has no GUI and you are completing WordPress configuration via a
web browser running on another computer.) Fill out the Site Title,
username, password, and E-mail and click Install WordPress.

Note the generated password (if applicable) and click the login
password. Your WordPress is now ready for use.

References
----------

-  `WordPress.org Codex <https://codex.wordpress.org/>`__

-  `Ubuntu Wiki
   WordPress <https://help.ubuntu.com/community/WordPress>`__


