Databases
=========

Ubuntu provides two popular database servers. They are:

-  MySQL

-  PostgreSQL

They are available in the main repository. This section explains how to
install and configure these database servers.

MySQL
=====

MySQL is a fast, multi-threaded, multi-user, and robust SQL database
server. It is intended for mission-critical, heavy-load production
systems as well as for embedding into mass-deployed software.

Installation
------------

To install MySQL, run the following command from a terminal prompt:

::

During the installation process you will be prompted to enter a password
for the MySQL root user.

Once the installation is complete, the MySQL server should be started
automatically. You can run the following command from a terminal prompt
to check whether the MySQL server is running:

::

When you run this command, you should see the following line or
something similar:

::

    tcp        0      0 localhost:mysql         *:*                LISTEN      2556/mysqld

If the server is not running correctly, you can type the following
command to start it:

::

Configuration
-------------

You can edit the ``/etc/mysql/my.cnf`` file to configure the basic
settings -- log file, port number, etc. For example, to configure MySQL
to listen for connections from network hosts, change the *bind-address*
directive to the server's IP address:

::

    bind-address            = 192.168.0.5

    **Note**

    Replace 192.168.0.5 with the appropriate address.

After making a change to ``/etc/mysql/my.cnf`` the MySQL daemon will
need to be restarted:

::

If you would like to change the MySQL *root* password, in a terminal
enter:

::

The MySQL daemon will be stopped, and you will be prompted to enter a
new password.

Database Engines
----------------

Whilst the default configuration of MySQL provided by the Ubuntu
packages is perfectly functional and performs well there are things you
may wish to consider before you proceed.

MySQL is designed to allow data to be stored in different ways. These
methods are referred to as either database or storage engines. There are
two main engines that you'll be interested in: InnoDB and MyISAM.
Storage engines are transparent to the end user. MySQL will handle
things differently under the surface, but regardless of which storage
engine is in use, you will interact with the database in the same way.

Each engine has its own advantages and disadvantages.

While it is possible, and may be advantageous to mix and match database
engines on a table level, doing so reduces the effectiveness of the
performance tuning you can do as you'll be splitting the resources
between two engines instead of dedicating them to one.

-  MyISAM is the older of the two. It can be faster than InnoDB under
   certain circumstances and favours a read only workload. Some web
   applications have been tuned around MyISAM (though that's not to
   imply that they will slow under InnoDB). MyISAM also supports the
   FULLTEXT data type, which allows very fast searches of large
   quantities of text data. However MyISAM is only capable of locking an
   entire table for writing. This means only one process can update a
   table at a time. As any application that uses the table scales this
   may prove to be a hindrance. It also lacks journaling, which makes it
   harder for data to be recovered after a crash. The following link
   provides some points for consideration about using `MyISAM on a
   production
   database <http://www.mysqlperformanceblog.com/2006/06/17/using-myisam-in-production/>`__.

-  InnoDB is a more modern database engine, designed to be `ACID
   compliant <http://en.wikipedia.org/wiki/ACID>`__ which guarantees
   database transactions are processed reliably. Write locking can occur
   on a row level basis within a table. That means multiple updates can
   occur on a single table simultaneously. Data caching is also handled
   in memory within the database engine, allowing caching on a more
   efficient row level basis rather than file block. To meet ACID
   compliance all transactions are journaled independently of the main
   tables. This allows for much more reliable data recovery as data
   consistency can be checked.

As of MySQL 5.5 InnoDB is the default engine, and is highly recommended
over MyISAM unless you have specific need for features unique to the
engine.

Advanced configuration
----------------------

Creating a tuned my.cnf file
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

There are a number of parameters that can be adjusted within MySQL's
configuration file that will allow you to improve the performance of the
server over time. For initial set-up you may find `Percona's my.cnf
generating tool <http://tools.percona.com/members/wizard>`__ useful.
This tool will help generate a my.cnf file that will be much more
optimised for your specific server capabilities and your requirements.

*Do not* replace your existing my.cnf file with Percona's one if you
have already loaded data into the database. Some of the changes that
will be in the file will be incompatible as they alter how data is
stored on the hard disk and you'll be unable to start MySQL. If you do
wish to use it and you have existing data, you will need to carry out a
mysqldump and reload:

::

    mysqldump --all-databases --routines -u root -p > ~/fulldump.sql

This will then prompt you for the root password before creating a copy
of the data. It is advisable to make sure there are no other users or
processes using the database whilst this takes place. Depending on how
much data you've got in your database, this may take a while. You won't
see anything on the screen during this process.

Once the dump has been completed, shut down MySQL:

::

Now backup the original my.cnf file and replace with the new one:

::


Then delete and re-initialise the database space and make sure ownership
is correct before restarting MySQL:

::




Finally all that's left is to re-import your data. To give us an idea of
how far the import process has got you may find the 'Pipe Viewer'
utility, pv, useful. The following shows how to install and use pv for
this case, but if you'd rather not use it just replace pv with cat in
the following command. Ignore any ETA times produced by pv, they're
based on the average time taken to handle each row of the file, but the
speed of inserting can vary wildly from row to row with mysqldumps:

::


Once that is complete all is good to go!

    **Note**

    This is not necessary for all my.cnf changes. Most of the variables
    you may wish to change to improve performance are adjustable even
    whilst the server is running. As with anything, make sure to have a
    good backup copy of config files and data before making changes.

MySQL Tuner
~~~~~~~~~~~

MySQL Tuner is a useful tool that will connect to a running MySQL
instance and offer suggestions for how it can be best configured for
your workload. The longer the server has been running for, the better
the advice mysqltuner can provide. In a production environment, consider
waiting for at least 24 hours before running the tool. You can get
install mysqltuner from the Ubuntu repositories:

::

Then once its been installed, run it:

::

and wait for its final report. The top section provides general
information about the database server, and the bottom section provides
tuning suggestions to alter in your my.cnf. Most of these can be altered
live on the server without restarting, look through the official MySQL
documentation (link in Resources section) for the relevant variables to
change in production. The following is part of an example report from a
production database which shows there may be some benefit from
increasing the amount of query cache:

::

    -------- Recommendations -----------------------------------------------------
    General recommendations:
        Run OPTIMIZE TABLE to defragment tables for better performance
        Increase table_cache gradually to avoid file descriptor limits
    Variables to adjust:
        key_buffer_size (> 1.4G)
        query_cache_size (> 32M)
        table_cache (> 64)
        innodb_buffer_pool_size (>= 22G)

One final comment on tuning databases: Whilst we can broadly say that
certain settings are the best, performance can vary from application to
application. For example, what works best for Wordpress might not be the
best for Drupal, Joomla or proprietary applications. Performance is
dependent on the types of queries, use of indexes, how efficient the
database design is and so on. You may find it useful to spend some time
searching for database tuning tips based on what applications you're
using it for. Once you get past a certain point any adjustments you make
will only result in minor improvements, and you'll be better off either
improving the application, or looking at scaling up your database
environment through either using more powerful hardware or by adding
slave servers.

Resources
---------

-  See the `MySQL Home Page <http://www.mysql.com/>`__ for more
   information.

-  Full documentation is available in both online and offline formats
   from the `MySQL Developers portal <http://dev.mysql.com/doc/>`__

-  For general SQL information see `Using SQL Special
   Edition <http://www.informit.com/store/product.aspx?isbn=0768664128>`__
   by Rafe Colburn.

-  The `Apache MySQL PHP Ubuntu
   Wiki <https://help.ubuntu.com/community/ApacheMySQLPHP>`__ page also
   has useful information.

PostgreSQL
==========

PostgreSQL is an object-relational database system that has the features
of traditional commercial database systems with enhancements to be found
in next-generation DBMS systems.

Installation
------------

To install PostgreSQL, run the following command in the command prompt:

::

Once the installation is complete, you should configure the PostgreSQL
server based on your needs, although the default configuration is
viable.

Configuration
-------------

PostgreSQL supports multiple client authentication methods. IDENT
authentication method is used for postgres and local users, unless
otherwise configured. Please refer to `the PostgreSQL Administrator's
Guide <http://www.postgresql.org/docs/9.1/static/admin.html>`__ if you
would like to configure alternatives like Kerberos.

The following discussion assumes that you wish to enable TCP/IP
connections and use the MD5 method for client authentication.
PostgreSQLconfiguration files are stored in the
``/etc/postgresql/<version>/main`` directory. For example, if you
install PostgreSQL 9.1, the configuration files are stored in the
``/etc/postgresql/9.1/main`` directory.

    **Tip**

    To configure *ident* authentication, add entries to the
    ``/etc/postgresql/9.1/main/pg_ident.conf`` file. There are detailed
    comments in the file to guide you.

To enable other computers to connect to your PostgreSQL server, edit the
file ``/etc/postgresql/9.1/main/postgresql.conf``

Locate the line *#listen\_addresses = 'localhost'* and change it to:

::

    listen_addresses = '*'

    **Note**

    To allow both IPv4 and IPv6 connections replace 'localhost' with
    '::'

You may also edit all other parameters, if you know what you are doing!
For details, refer to the configuration file or to the PostgreSQL
documentation.

Now that we can connect to our PostgreSQL server, the next step is to
set a password for the *postgres* user. Run the following command at a
terminal prompt to connect to the default PostgreSQL template database:

::

The above command connects to PostgreSQL database *template1* as user
*postgres*. Once you connect to the PostgreSQL server, you will be at a
SQL prompt. You can run the following SQL command at the psql prompt to
configure the password for the user *postgres*.

::

After configuring the password, edit the file
``/etc/postgresql/9.1/main/pg_hba.conf`` to use *MD5* authentication
with the *postgres* user:

::

    local   all         postgres                          md5

Finally, you should restart the PostgreSQL service to initialize the new
configuration. From a terminal prompt enter the following to restart
PostgreSQL:

::

    **Warning**

    The above configuration is not complete by any means. Please refer
    `the PostgreSQL Administrator's
    Guide <http://www.postgresql.org/docs/9.1/static/admin.html>`__ to
    configure more parameters.

You can test server connections from other machines by using the
PostgreSQL client.

::


    **Note**

    Replace the domain name with your actual server domain name.

Backups
-------

PostgreSQL databases should be backed up regularly. Refer to the `the
PostgreSQL Administrator's
Guide <http://www.postgresql.org/docs/9.1/static/backup.html>`__ for
different approaches.

Resources
---------

-  As mentioned above the `the PostgreSQL Administrator's
   Guide <http://www.postgresql.org/docs/9.1/static/admin.html>`__ is an
   excellent resource. The guide is also available in the
   postgresql-doc-9.1 package. Execute the following in a terminal to
   install the package:

   ::

   To view the guide enter
   ``file:///usr/share/doc/postgresql-doc-9.1/html/index.html`` into the
   address bar of your browser.

-  For general SQL information see `Using SQL Special
   Edition <http://www.informit.com/store/product.aspx?isbn=0768664128>`__
   by Rafe Colburn.

-  Also, see the `PostgreSQL Ubuntu
   Wiki <https://help.ubuntu.com/community/PostgreSQL>`__ page for more
   information.


