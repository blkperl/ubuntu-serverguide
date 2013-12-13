Web Servers
===========

A Web server is a software responsible for accepting HTTP requests from
clients, which are known as Web browsers, and serving them HTTP
responses along with optional data contents, which usually are Web pages
such as HTML documents and linked objects (images, etc.).

HTTPD - Apache2 Web Server
==========================

Apache is the most commonly used Web Server on Linux systems. Web
Servers are used to serve Web Pages requested by client computers.
Clients typically request and view Web Pages using Web Browser
applications such as Firefox, Opera, Chromium, or Mozilla.

Users enter a Uniform Resource Locator (URL) to point to a Web server by
means of its Fully Qualified Domain Name (FQDN) and a path to the
required resource. For example, to view the home page of the `Ubuntu Web
site <&ubuntu-web;>`__ a user will enter only the FQDN:

::

To view the `community <http://www.ubuntu.com/community>`__ sub-page, a
user will enter the FQDN followed by a path:

::

The most common protocol used to transfer Web pages is the Hyper Text
Transfer Protocol (HTTP). Protocols such as Hyper Text Transfer Protocol
over Secure Sockets Layer (HTTPS), and File Transfer Protocol (FTP), a
protocol for uploading and downloading files, are also supported.

Apache Web Servers are often used in combination with the MySQL database
engine, the HyperText Preprocessor (PHP) scripting language, and other
popular scripting languages such as Python and Perl. This configuration
is termed LAMP (Linux, Apache, MySQL and Perl/Python/PHP) and forms a
powerful and robust platform for the development and deployment of
Web-based applications.

Installation
------------

The Apache2 web server is available in Ubuntu Linux. To install Apache2:

At a terminal prompt enter the following command:

::

Configuration
-------------

Apache2 is configured by placing *directives* in plain text
configuration files. These *directives* are separated between the
following files and directories:

-  *apache2.conf:* the main Apache2 configuration file. Contains
   settings that are *global* to Apache2.

-  *conf.d:* contains configuration files which apply *globally* to
   Apache2. Other packages that use Apache2 to serve content may add
   files, or symlinks, to this directory.

-  *envvars:* file where Apache2 *environment* variables are set.

-  *httpd.conf:* historically the main Apache2 configuration file, named
   after the httpd daemon. Now the file does not exist. In older
   versions of Ubuntu the file might be present, but empty, as all
   configuration options have been moved to the below referenced
   directories.

-  *mods-available:* this directory contains configuration files to both
   load *modules* and configure them. Not all modules will have specific
   configuration files, however.

-  *mods-enabled:* holds *symlinks* to the files in
   ``/etc/apache2/mods-available``. When a module configuration file is
   symlinked it will be enabled the next time apache2 is restarted.

-  *ports.conf:* houses the directives that determine which TCP ports
   Apache2 is listening on.

-  *sites-available:* this directory has configuration files for Apache2
   *Virtual Hosts*. Virtual Hosts allow Apache2 to be configured for
   multiple sites that have separate configurations.

-  *sites-enabled:* like mods-enabled, ``sites-enabled`` contains
   symlinks to the ``/etc/apache2/sites-available`` directory. Similarly
   when a configuration file in sites-available is symlinked, the site
   configured by it will be active once Apache2 is restarted.

In addition, other configuration files may be added using the *Include*
directive, and wildcards can be used to include many configuration
files. Any directive may be placed in any of these configuration files.
Changes to the main configuration files are only recognized by Apache2
when it is started or restarted.

The server also reads a file containing mime document types; the
filename is set by the *TypesConfig* directive, typically via
``/etc/apache2/mods-available/mime.conf``, which might also include
additions and overrides, and is ``/etc/mime.types`` by default.

Basic Settings
~~~~~~~~~~~~~~

This section explains Apache2 server essential configuration parameters.
Refer to the `Apache2
Documentation <http://httpd.apache.org/docs/2.2/>`__ for more details.

-  Apache2 ships with a virtual-host-friendly default configuration.
   That is, it is configured with a single default virtual host (using
   the *VirtualHost* directive) which can modified or used as-is if you
   have a single site, or used as a template for additional virtual
   hosts if you have multiple sites. If left alone, the default virtual
   host will serve as your default site, or the site users will see if
   the URL they enter does not match the *ServerName* directive of any
   of your custom sites. To modify the default virtual host, edit the
   file ``/etc/apache2/sites-available/default``.

       **Note**

       The directives set for a virtual host only apply to that
       particular virtual host. If a directive is set server-wide and
       not defined within the virtual host settings, the default setting
       is used. For example, you can define a Webmaster email address
       and not define individual email addresses for each virtual host.

   If you wish to configure a new virtual host or site, copy that file
   into the same directory with a name you choose. For example:

   ::

   Edit the new file to configure the new site using some of the
   directives described below.

-  The *ServerAdmin* directive specifies the email address to be
   advertised for the server's administrator. The default value is
   webmaster@localhost. This should be changed to an email address that
   is delivered to you (if you are the server's administrator). If your
   website has a problem, Apache2 will display an error message
   containing this email address to report the problem to. Find this
   directive in your site's configuration file in
   /etc/apache2/sites-available.

-  The *Listen* directive specifies the port, and optionally the IP
   address, Apache2 should listen on. If the IP address is not
   specified, Apache2 will listen on all IP addresses assigned to the
   machine it runs on. The default value for the Listen directive is 80.
   Change this to 127.0.0.1:80 to cause Apache2 to listen only on your
   loopback interface so that it will not be available to the Internet,
   to (for example) 81 to change the port that it listens on, or leave
   it as is for normal operation. This directive can be found and
   changed in its own file, ``/etc/apache2/ports.conf``

-  The *ServerName* directive is optional and specifies what FQDN your
   site should answer to. The default virtual host has no ServerName
   directive specified, so it will respond to all requests that do not
   match a ServerName directive in another virtual host. If you have
   just acquired the domain name ubunturocks.com and wish to host it on
   your Ubuntu server, the value of the ServerName directive in your
   virtual host configuration file should be ubunturocks.com. Add this
   directive to the new virtual host file you created earlier
   (``/etc/apache2/sites-available/mynewsite``).

   You may also want your site to respond to www.ubunturocks.com, since
   many users will assume the www prefix is appropriate. Use the
   *ServerAlias* directive for this. You may also use wildcards in the
   ServerAlias directive.

   For example, the following configuration will cause your site to
   respond to any domain request ending in *.ubunturocks.com*.

   ::

       ServerAlias *.ubunturocks.com

-  The *DocumentRoot* directive specifies where Apache2 should look for
   the files that make up the site. The default value is /var/www, as
   specified in ``/etc/apache2/sites-available/default``. If desired,
   change this value in your site's virtual host file, and remember to
   create that directory if necessary!

Enable the new *VirtualHost* using the a2ensite utility and restart
Apache2:

::


    **Note**

    Be sure to replace *mynewsite* with a more descriptive name for the
    VirtualHost. One method is to name the file after the *ServerName*
    directive of the VirtualHost.

Similarly, use the a2dissite utility to disable sites. This is can be
useful when troubleshooting configuration problems with multiple
VirtualHosts:

::


Default Settings
~~~~~~~~~~~~~~~~

This section explains configuration of the Apache2 server default
settings. For example, if you add a virtual host, the settings you
configure for the virtual host take precedence for that virtual host.
For a directive not defined within the virtual host settings, the
default value is used.

-  The *DirectoryIndex* is the default page served by the server when a
   user requests an index of a directory by specifying a forward slash
   (/) at the end of the directory name.

   For example, when a user requests the page
   http://www.example.com/this\_directory/, he or she will get either
   the DirectoryIndex page if it exists, a server-generated directory
   list if it does not and the Indexes option is specified, or a
   Permission Denied page if neither is true. The server will try to
   find one of the files listed in the DirectoryIndex directive and will
   return the first one it finds. If it does not find any of these files
   and if *Options Indexes* is set for that directory, the server will
   generate and return a list, in HTML format, of the subdirectories and
   files in the directory. The default value, found in
   ``/etc/apache2/mods-available/dir.conf`` is "index.html index.cgi
   index.pl index.php index.xhtml index.htm". Thus, if Apache2 finds a
   file in a requested directory matching any of these names, the first
   will be displayed.

-  The *ErrorDocument* directive allows you to specify a file for
   Apache2 to use for specific error events. For example, if a user
   requests a resource that does not exist, a 404 error will occur. By
   default, Apache2 will simply return a HTTP 404 Return code. Read
   ``/etc/apache2/conf.d/localized-error-pages`` for detailed
   instructions for using ErrorDocument, including locations of example
   files.

-  By default, the server writes the transfer log to the file
   ``/var/log/apache2/access.log``. You can change this on a per-site
   basis in your virtual host configuration files with the *CustomLog*
   directive, or omit it to accept the default, specified in ``
             /etc/apache2/conf.d/other-vhosts-access-log``. You may also
   specify the file to which errors are logged, via the *ErrorLog*
   directive, whose default is ``/var/log/apache2/error.log``. These are
   kept separate from the transfer logs to aid in troubleshooting
   problems with your Apache2 server. You may also specify the
   *LogLevel* (the default value is "warn") and the *LogFormat* (see ``
             /etc/apache2/apache2.conf`` for the default value).

-  Some options are specified on a per-directory basis rather than
   per-server. *Options* is one of these directives. A Directory stanza
   is enclosed in XML-like tags, like so:

   ::

       <Directory /var/www/mynewsite>
       ...
       </Directory>

   The *Options* directive within a Directory stanza accepts one or more
   of the following values (among others), separated by spaces:

   -  **ExecCGI** - Allow execution of CGI scripts. CGI scripts are not
      executed if this option is not chosen.

          **Tip**

          Most files should not be executed as CGI scripts. This would
          be very dangerous. CGI scripts should kept in a directory
          separate from and outside your DocumentRoot, and only this
          directory should have the ExecCGI option set. This is the
          default, and the default location for CGI scripts is
          ``/usr/lib/cgi-bin``.

   -  **Includes** - Allow server-side includes. Server-side includes
      allow an HTML file to *include* other files. See `Apache SSI
      documentation (Ubuntu
      community) <https://help.ubuntu.com/community/ServerSideIncludes>`__
      for more information.

   -  **IncludesNOEXEC** - Allow server-side includes, but disable the
      *#exec* and *#include* commands in CGI scripts.

   -  **Indexes** - Display a formatted list of the directory's
      contents, if no *DirectoryIndex* (such as index.html) exists in
      the requested directory.

          **Caution**

          For security reasons, this should usually not be set, and
          certainly should not be set on your DocumentRoot directory.
          Enable this option carefully on a per-directory basis only if
          you are certain you want users to see the entire contents of
          the directory.

   -  **Multiview** - Support content-negotiated multiviews; this option
      is disabled by default for security reasons. See the `Apache2
      documentation on this
      option <http://httpd.apache.org/docs/2.2/mod/mod_negotiation.html#multiviews>`__.

   -  **SymLinksIfOwnerMatch** - Only follow symbolic links if the
      target file or directory has the same owner as the link.

httpd Settings
~~~~~~~~~~~~~~

This section explains some basic httpd daemon configuration settings.

**LockFile** - The LockFile directive sets the path to the lockfile used
when the server is compiled with either USE\_FCNTL\_SERIALIZED\_ACCEPT
or USE\_FLOCK\_SERIALIZED\_ACCEPT. It must be stored on the local disk.
It should be left to the default value unless the logs directory is
located on an NFS share. If this is the case, the default value should
be changed to a location on the local disk and to a directory that is
readable only by root.

**PidFile** - The PidFile directive sets the file in which the server
records its process ID (pid). This file should only be readable by root.
In most cases, it should be left to the default value.

**User** - The User directive sets the userid used by the server to
answer requests. This setting determines the server's access. Any files
inaccessible to this user will also be inaccessible to your website's
visitors. The default value for User is "www-data".

    **Warning**

    Unless you know exactly what you are doing, do not set the User
    directive to root. Using root as the User will create large security
    holes for your Web server.

**Group** - The Group directive is similar to the User directive. Group
sets the group under which the server will answer requests. The default
group is also "www-data".

Apache2 Modules
~~~~~~~~~~~~~~~

Apache2 is a modular server. This implies that only the most basic
functionality is included in the core server. Extended features are
available through modules which can be loaded into Apache2. By default,
a base set of modules is included in the server at compile-time. If the
server is compiled to use dynamically loaded modules, then modules can
be compiled separately, and added at any time using the LoadModule
directive. Otherwise, Apache2 must be recompiled to add or remove
modules.

Ubuntu compiles Apache2 to allow the dynamic loading of modules.
Configuration directives may be conditionally included on the presence
of a particular module by enclosing them in an *<IfModule>* block.

You can install additional Apache2 modules and use them with your Web
server. For example, run the following command from a terminal prompt to
install the *MySQL Authentication* module:

::

See the ``/etc/apache2/mods-available`` directory, for additional
modules.

Use the a2enmod utility to enable a module:

::


Similarly, a2dismod will disable a module:

::


HTTPS Configuration
-------------------

The mod\_ssl module adds an important feature to the Apache2 server -
the ability to encrypt communications. Thus, when your browser is
communicating using SSL, the https:// prefix is used at the beginning of
the Uniform Resource Locator (URL) in the browser navigation bar.

The mod\_ssl module is available in apache2-common package. Execute the
following command from a terminal prompt to enable the mod\_ssl module:

::

There is a default HTTPS configuration file in
``/etc/apache2/sites-available/default-ssl``. In order for Apache2 to
provide HTTPS, a *certificate* and *key* file are also needed. The
default HTTPS configuration will use a certificate and key generated by
the ssl-cert package. They are good for testing, but the auto-generated
certificate and key should be replaced by a certificate specific to the
site or server. For information on generating a key and obtaining a
certificate see ?

To configure Apache2 for HTTPS, enter the following:

::

    **Note**

    The directories ``/etc/ssl/certs`` and ``/etc/ssl/private`` are the
    default locations. If you install the certificate and key in another
    directory make sure to change *SSLCertificateFile* and
    *SSLCertificateKeyFile* appropriately.

With Apache2 now configured for HTTPS, restart the service to enable the
new settings:

::

    **Note**

    Depending on how you obtained your certificate you may need to enter
    a passphrase when Apache2 starts.

You can access the secure server pages by typing
https://your\_hostname/url/ in your browser address bar.

Sharing Write Permission
------------------------

For more than one user to be able to write to the same directory it will
be necessary to grant write permission to a group they share in common.
The following example grants shared write permission to ``/var/www`` to
the group "webmasters".

::



    **Note**

    If access must be granted to more than one group per directory,
    enable Access Control Lists (ACLs).

References
----------

-  `Apache2 Documentation <http://httpd.apache.org/docs/2.2/>`__
   contains in depth information on Apache2 configuration directives.
   Also, see the apache2-doc package for the official Apache2 docs.

-  See the `Mod SSL Documentation <http://www.modssl.org/docs/>`__ site
   for more SSL related information.

-  O'Reilly's `Apache
   Cookbook <http://oreilly.com/catalog/9780596001919/>`__ is a good
   resource for accomplishing specific Apache2 configurations.

-  For Ubuntu specific Apache2 questions, ask in the *#ubuntu-server*
   IRC channel on `freenode.net <http://freenode.net/>`__.

-  Usually integrated with PHP and MySQL the `Apache MySQL PHP Ubuntu
   Wiki <https://help.ubuntu.com/community/ApacheMySQLPHP>`__ page is a
   good resource.

PHP5 - Scripting Language
=========================

PHP is a general-purpose scripting language suited for Web development.
The PHP script can be embedded into HTML. This section explains how to
install and configure PHP5 in Ubuntu System with Apache2 and MySQL.

This section assumes you have installed and configured Apache2 Web
Server and MySQL Database Server. You can refer to Apache2 section and
MySQL sections in this document to install and configure Apache2 and
MySQL respectively.

Installation
------------

The PHP5 is available in Ubuntu Linux. Unlike python and perl, which are
installed in the base system, PHP must be added.

To install PHP5 you can enter the following command in the terminal
prompt:

::

You can run PHP5 scripts from command line. To run PHP5 scripts from
command line you should install php5-cli package. To install php5-cli
you can enter the following command in the terminal prompt:

::

You can also execute PHP5 scripts without installing PHP5 Apache module.
To accomplish this, you should install php5-cgi package. You can run the
following command in a terminal prompt to install php5-cgi package:

::

To use MySQL with PHP5 you should install php5-mysql package. To install
php5-mysql you can enter the following command in the terminal prompt:

::

Similarly, to use PostgreSQL with PHP5 you should install php5-pgsql
package. To install php5-pgsql you can enter the following command in
the terminal prompt:

::

Configuration
-------------

Once you install PHP5, you can run PHP5 scripts from your web browser.
If you have installed php5-cli package, you can run PHP5 scripts from
your command prompt.

By default, the Apache 2 Web server is configured to run PHP5 scripts.
In other words, the PHP5 module is enabled in Apache2 Web server
automatically when you install the module. Please verify if the files
``/etc/apache2/mods-enabled/php5.conf`` and
``/etc/apache2/mods-enabled/php5.load`` exist. If they do not exists,
you can enable the module using ``a2enmod`` command.

Once you install PHP5 related packages and enabled PHP5 Apache 2 module,
you should restart Apache2 Web server to run PHP5 scripts. You can run
the following command at a terminal prompt to restart your web server:

::

     

Testing
-------

To verify your installation, you can run following PHP5 phpinfo script:

::

    <?php
      phpinfo();
    ?>

You can save the content in a file ``phpinfo.php`` and place it under
``DocumentRoot`` directory of Apache2 Web server. When point your
browser to ``http://hostname/phpinfo.php``, it would display values of
various PHP5 configuration parameters.

References
----------

-  For more in depth information see
   `php.net <http://www.php.net/docs.php>`__ documentation.

-  There are a plethora of books on PHP. Two good books from O'Reilly
   are `Learning PHP 5 <http://oreilly.com/catalog/9780596005603/>`__
   and the `PHP Cook
   Book <http://oreilly.com/catalog/9781565926813/>`__.

-  Also, see the `Apache MySQL PHP Ubuntu
   Wiki <https://help.ubuntu.com/community/ApacheMySQLPHP>`__ page for
   more information.

Squid - Proxy Server
====================

Squid is a full-featured web proxy cache server application which
provides proxy and cache services for Hyper Text Transport Protocol
(HTTP), File Transfer Protocol (FTP), and other popular network
protocols. Squid can implement caching and proxying of Secure Sockets
Layer (SSL) requests and caching of Domain Name Server (DNS) lookups,
and perform transparent caching. Squid also supports a wide variety of
caching protocols, such as Internet Cache Protocol (ICP), the Hyper Text
Caching Protocol (HTCP), the Cache Array Routing Protocol (CARP), and
the Web Cache Coordination Protocol (WCCP).

The Squid proxy cache server is an excellent solution to a variety of
proxy and caching server needs, and scales from the branch office to
enterprise level networks while providing extensive, granular access
control mechanisms, and monitoring of critical parameters via the Simple
Network Management Protocol (SNMP). When selecting a computer system for
use as a dedicated Squid caching proxy server for many users ensure it
is configured with a large amount of physical memory as Squid maintains
an in-memory cache for increased performance.

Installation
------------

At a terminal prompt, enter the following command to install the Squid
server:

::

Configuration
-------------

Squid is configured by editing the directives contained within the
``/etc/squid3/squid.conf`` configuration file. The following examples
illustrate some of the directives which may be modified to affect the
behavior of the Squid server. For more in-depth configuration of Squid,
see the References section.

    **Tip**

    Prior to editing the configuration file, you should make a copy of
    the original file and protect it from writing so you will have the
    original settings as a reference, and to re-use as necessary. Make
    this copy and protect it from writing using the following commands:

    ::


-  To set your Squid server to listen on TCP port 8888 instead of the
   default TCP port 3128, change the http\_port directive as such:

   ::

       http_port 8888

-  Change the visible\_hostname directive in order to give the Squid
   server a specific hostname. This hostname does not necessarily need
   to be the computer's hostname. In this example it is set to *weezie*

   ::

       visible_hostname weezie

-  Using Squid's access control, you may configure use of Internet
   services proxied by Squid to be available only users with certain
   Internet Protocol (IP) addresses. For example, we will illustrate
   access by users of the 192.168.42.0/24 subnetwork only:

   Add the following to the **bottom** of the ACL section of your
   ``/etc/squid3/squid.conf`` file:

   ::

       acl fortytwo_network src 192.168.42.0/24

   Then, add the following to the **top** of the http\_access section of
   your ``/etc/squid3/squid.conf`` file:

   ::

       http_access allow fortytwo_network

-  Using the excellent access control features of Squid, you may
   configure use of Internet services proxied by Squid to be available
   only during normal business hours. For example, we'll illustrate
   access by employees of a business which is operating between 9:00AM
   and 5:00PM, Monday through Friday, and which uses the 10.1.42.0/24
   subnetwork:

   Add the following to the **bottom** of the ACL section of your
   ``/etc/squid3/squid.conf`` file:

   ::

       acl biz_network src 10.1.42.0/24
       acl biz_hours time M T W T F 9:00-17:00

   Then, add the following to the **top** of the http\_access section of
   your ``/etc/squid3/squid.conf`` file:

   ::

       http_access allow biz_network biz_hours

    **Note**

    After making changes to the ``/etc/squid3/squid.conf`` file, save
    the file and restart the squid server application to effect the
    changes using the following command entered at a terminal prompt:

    ::

References
----------

`Squid Website <http://www.squid-cache.org/>`__

`Ubuntu Wiki Squid <https://help.ubuntu.com/community/Squid>`__ page.

Ruby on Rails
=============

Ruby on Rails is an open source web framework for developing database
backed web applications. It is optimized for sustainable productivity of
the programmer since it lets the programmer to write code by favouring
convention over configuration.

Installation
------------

Before installing Rails you should install Apache and MySQL. To install
the Apache package, please refer to ?. For instructions on installing
MySQL refer to ?.

Once you have Apache and MySQL packages installed, you are ready to
install Ruby on Rails package.

To install the Ruby base packages and Ruby on Rails, you can enter the
following command in the terminal prompt:

::

Configuration
-------------

Modify the ``/etc/apache2/sites-available/default`` configuration file
to setup your domains.

The first thing to change is the *DocumentRoot* directive:

::

    DocumentRoot /path/to/rails/application/public

Next, change the <Directory "/path/to/rails/application/public">
directive:

::

    <Directory "/path/to/rails/application/public">
            Options Indexes FollowSymLinks MultiViews ExecCGI
            AllowOverride All
            Order allow,deny
            allow from all
            AddHandler cgi-script .cgi
    </Directory>

You should also enable the mod\_rewrite module for Apache. To enable
mod\_rewrite module, please enter the following command in a terminal
prompt:

::

Finally you will need to change the ownership of the
``/path/to/rails/application/public`` and
``/path/to/rails/application/tmp`` directories to the user used to run
the Apache process:

::


That's it! Now you have your Server ready for your Ruby on Rails
applications.

References
----------

-  See the `Ruby on Rails <http://rubyonrails.org/>`__ website for more
   information.

-  Also `Agile Development with
   Rails <http://pragprog.com/titles/rails3/agile-web-development-with-rails-third-edition>`__
   is a great resource.

-  Another place for more information is the `Ruby on Rails Ubuntu
   Wiki <https://help.ubuntu.com/community/RubyOnRails>`__ page.

Apache Tomcat
=============

Apache Tomcat is a web container that allows you to serve Java Servlets
and JSP (Java Server Pages) web applications.

Ubuntu has supported packages for both Tomcat 6 and 7. Tomcat 6 is the
legacy version, and Tomcat 7 is the current version where new features
are implemented. Both are considered stable. This guide will focus on
Tomcat 7, but most configuration details are valid for both versions.

The Tomcat packages in Ubuntu support two different ways of running
Tomcat. You can install them as a classic unique system-wide instance,
that will be started at boot time will run as the tomcat7 (or tomcat6)
unprivileged user. But you can also deploy private instances that will
run with your own user rights, and that you should start and stop by
yourself. This second way is particularly useful in a development server
context where multiple users need to test on their own private Tomcat
instances.

System-wide installation
------------------------

To install the Tomcat server, you can enter the following command in the
terminal prompt:

::

This will install a Tomcat server with just a default ROOT webapp that
displays a minimal "It works" page by default.

Configuration
-------------

Tomcat configuration files can be found in ``/etc/tomcat7``. Only a few
common configuration tweaks will be described here, please see `Tomcat
7.0
documentation <http://tomcat.apache.org/tomcat-7.0-doc/index.html>`__
for more.

Changing default ports
~~~~~~~~~~~~~~~~~~~~~~

By default Tomcat runs a HTTP connector on port 8080 and an AJP
connector on port 8009. You might want to change those default ports to
avoid conflict with another application on the system. This is done by
changing the following lines in ``/etc/tomcat7/server.xml``:

::

    <Connector port="8080" protocol="HTTP/1.1" 
                   connectionTimeout="20000" 
                   redirectPort="8443" />
    ...
    <Connector port="8009" protocol="AJP/1.3" redirectPort="8443" />

Changing JVM used
~~~~~~~~~~~~~~~~~

By default Tomcat will run preferably with OpenJDK JVMs, then try the
Sun JVMs, then try some other JVMs. You can force Tomcat to use a
specific JVM by setting JAVA\_HOME in ``/etc/default/tomcat7``:

::

    JAVA_HOME=/usr/lib/jvm/java-6-sun

Declaring users and roles
~~~~~~~~~~~~~~~~~~~~~~~~~

Usernames, passwords and roles (groups) can be defined centrally in a
Servlet container. This is done in the ``/etc/tomcat7/tomcat-users.xml``
file:

::

    <role rolename="admin"/>
    <user username="tomcat" password="s3cret" roles="admin"/>

Using Tomcat standard webapps
-----------------------------

Tomcat is shipped with webapps that you can install for documentation,
administration or demo purposes.

Tomcat documentation
~~~~~~~~~~~~~~~~~~~~

The tomcat7-docs package contains Tomcat documentation, packaged as a
webapp that you can access by default at http://yourserver:8080/docs.
You can install it by entering the following command in the terminal
prompt:

::

Tomcat administration webapps
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The tomcat7-admin package contains two webapps that can be used to
administer the Tomcat server using a web interface. You can install them
by entering the following command in the terminal prompt:

::

The first one is the *manager* webapp, which you can access by default
at http://yourserver:8080/manager/html. It is primarily used to get
server status and restart webapps.

    **Note**

    Access to the *manager* application is protected by default: you
    need to define a user with the role "manager-gui" in
    ``/etc/tomcat7/tomcat-users.xml`` before you can access it.

The second one is the *host-manager* webapp, which you can access by
default at http://yourserver:8080/host-manager/html. It can be used to
create virtual hosts dynamically.

    **Note**

    Access to the *host-manager* application is also protected by
    default: you need to define a user with the role "admin-gui" in
    ``/etc/tomcat7/tomcat-users.xml`` before you can access it.

For security reasons, the tomcat7 user cannot write to the
``/etc/tomcat7`` directory by default. Some features in these admin
webapps (application deployment, virtual host creation) need write
access to that directory. If you want to use these features execute the
following, to give users in the tomcat7 group the necessary rights:

::


     

Tomcat examples webapps
~~~~~~~~~~~~~~~~~~~~~~~

The tomcat7-examples package contains two webapps that can be used to
test or demonstrate Servlets and JSP features, which you can access them
by default at http://yourserver:8080/examples. You can install them by
entering the following command in the terminal prompt:

::

Using private instances
-----------------------

Tomcat is heavily used in development and testing scenarios where using
a single system-wide instance doesn't meet the requirements of multiple
users on a single system. The Tomcat packages in Ubuntu come with tools
to help deploy your own user-oriented instances, allowing every user on
a system to run (without root rights) separate private instances while
still using the system-installed libraries.

    **Note**

    It is possible to run the system-wide instance and the private
    instances in parallel, as long as they do not use the same TCP
    ports.

Installing private instance support
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

You can install everything necessary to run private instances by
entering the following command in the terminal prompt:

::

Creating a private instance
~~~~~~~~~~~~~~~~~~~~~~~~~~~

You can create a private instance directory by entering the following
command in the terminal prompt:

::

This will create a new ``my-instance`` directory with all the necessary
subdirectories and scripts. You can for example install your common
libraries in the ``lib/`` subdirectory and deploy your webapps in the
``webapps/`` subdirectory. No webapps are deployed by default.

Configuring your private instance
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

You will find the classic Tomcat configuration files for your private
instance in the ``conf/`` subdirectory. You should for example certainly
edit the ``conf/server.xml`` file to change the default ports used by
your private Tomcat instance to avoid conflict with other instances that
might be running.

Starting/stopping your private instance
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

You can start your private instance by entering the following command in
the terminal prompt (supposing your instance is located in the
``my-instance`` directory):

::

    **Note**

    You should check the ``logs/`` subdirectory for any error. If you
    have a *java.net.BindException: Address already in use<null>:8080*
    error, it means that the port you're using is already taken and that
    you should change it.

You can stop your instance by entering the following command in the
terminal prompt (supposing your instance is located in the
``my-instance`` directory):

::

References
----------

-  See the `Apache Tomcat <http://tomcat.apache.org/>`__ website for
   more information.

-  `Tomcat: The Definitive
   Guide <http://shop.oreilly.com/product/9780596003180.do>`__ is a good
   resource for building web applications with Tomcat.

-  For additional books see the `Tomcat
   Books <http://wiki.apache.org/tomcat/Tomcat/Books>`__ list page.


