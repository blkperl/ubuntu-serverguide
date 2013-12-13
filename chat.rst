Chat Applications
=================

Overview
========

In this section, we will discuss how to install and configure a IRC
server, ircd-irc2. We will also discuss how to install and configure
Jabber, an instance messaging server.

IRC Server
==========

The Ubuntu repository has many Internet Relay Chat servers. This section
explains how to install and configure the original IRC server ircd-irc2.

Installation
------------

To install ircd-irc2, run the following command in the command prompt:

::

The configuration files are stored in ``/etc/ircd`` directory. The
documents are available in ``/usr/share/doc/ircd-irc2`` directory.

Configuration
-------------

The IRC settings can be done in the configuration file
``/etc/ircd/ircd.conf``. You can set the IRC host name in this file by
editing the following line:

::

    M:irc.localhost::Debian ircd default configuration::000A

Please make sure you add DNS aliases for the IRC host name. For
instance, if you set irc.livecipher.com as IRC host name, please make
sure irc.livecipher.com is resolvable in your Domain Name Server. The
IRC host name should not be same as the host name.

The IRC admin details can be configured by editing the following line:

::

    A:Organization, IRC dept.:Daemon <ircd@example.irc.org>:Client Server::IRCnet:

You should add specific lines to configure the list of IRC ports to
listen on, to configure Operator credentials, to configure client
authentication, etc. For details, please refer to the example
configuration file ``/usr/share/doc/ircd-irc2/ircd.conf.example.gz``.

The IRC banner to be displayed in the IRC client, when the user connects
to the server can be set in ``/etc/ircd/ircd.motd`` file.

After making necessary changes to the configuration file, you can
restart the IRC server using following command:

::

    sudo service ircd-irc2 restart

References
----------

You may also be interested to take a look at other IRC servers available
in Ubuntu Repository. It includes, ircd-ircu and ircd-hybrid.

-  Refer to `IRCD FAQ <http://www.irc.org/tech_docs/ircnet/faq.html>`__
   for more details about the IRC Server.

Jabber Instant Messaging Server
===============================

*Jabber* a popular instant message protocol is based on XMPP, an open
standard for instant messaging, and used by many popular applications.
This section covers setting up a *Jabberd 2* server on a local LAN. This
configuration can also be adapted to providing messaging services to
users over the Internet.

Installation
------------

To install jabberd2, in a terminal enter:

::

Configuration
-------------

A couple of XML configuration files will be used to configure jabberd2
for *Berkeley DB* user authentication. This is a very simple form of
authentication. However, jabberd2 can be configured to use LDAP, MySQL,
PostgreSQL, etc for for user authentication.

First, edit ``/etc/jabberd2/sm.xml`` changing:

::

      <id>jabber.example.com</id>

    **Note**

    Replace *jabber.example.com* with the hostname, or other id, of your
    server.

Now in the <storage> section change the <driver> to:

::

       <driver>db</driver>

Next, edit ``/etc/jabberd2/c2s.xml`` in the *<local>* section change:

::

        <id>jabber.example.com</id>

And in the <authreg> section adjust the <module> section to:

::

        <module>db</module>

Finally, restart jabberd2 to enable the new settings:

::

You should now be able to connect to the server using a Jabber client
like Pidgin for example.

    **Note**

    The advantage of using Berkeley DB for user data is that after being
    configured no additional maintenance is required. If you need more
    control over user accounts and credentials another authentication
    method is recommended.

References
----------

-  The `Jabberd2 Web
   Site <http://codex.xiaoka.com/wiki/jabberd2:start>`__ contains more
   details on configuring Jabberd2.

-  For more authentication options see the `Jabberd2 Install
   Guide <http://www.jabberdoc.org/>`__.

-  Also, the `Setting Up Jabber Server Ubuntu
   Wiki <https://help.ubuntu.com/community/SettingUpJabberServer>`__
   page has more information.


