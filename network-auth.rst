Network Authentication
======================

This section applies LDAP to network authentication and authorization.

OpenLDAP Server
===============

The Lightweight Directory Access Protocol, or LDAP, is a protocol for
querying and modifying a X.500-based directory service running over
TCP/IP. The current LDAP version is LDAPv3, as defined in
`RFC4510 <http://tools.ietf.org/html/rfc4510>`__, and the LDAP
implementation used in Ubuntu is OpenLDAP, currently at version 2.4.25
(Oneiric).

So this protocol accesses LDAP directories. Here are some key concepts
and terms:

-  A LDAP directory is a tree of data *entries* that is hierarchical in
   nature and is called the Directory Information Tree (DIT).

-  An entry consists of a set of *attributes*.

-  An attribute has a *type* (a name/description) and one or more
   *values*.

-  Every attribute must be defined in at least one *objectClass*.

-  Attributes and objectclasses are defined in *schemas* (an objectclass
   is actually considered as a special kind of attribute).

-  Each entry has a unique identifier: it's *Distinguished Name* (DN or
   dn). This consists of it's *Relative Distinguished Name* (RDN)
   followed by the parent entry's DN.

-  The entry's DN is not an attribute. It is not considered part of the
   entry itself.

    **Note**

    The terms *object*, *container*, and *node* have certain
    connotations but they all essentially mean the same thing as
    *entry*, the technically correct term.

For example, below we have a single entry consisting of 11 attributes.
It's DN is "cn=John Doe,dc=example,dc=com"; it's RDN is "cn=John Doe";
and it's parent DN is "dc=example,dc=com".

::

     dn: cn=John Doe,dc=example,dc=com
     cn: John Doe
     givenName: John
     sn: Doe
     telephoneNumber: +1 888 555 6789
     telephoneNumber: +1 888 555 1232
     mail: john@example.com
     manager: cn=Larry Smith,dc=example,dc=com
     objectClass: inetOrgPerson
     objectClass: organizationalPerson
     objectClass: person
     objectClass: top

The above entry is in *LDIF* format (LDAP Data Interchange Format). Any
information that you feed into your DIT must also be in such a format.
It is defined in `RFC2849 <http://tools.ietf.org/html/rfc2849>`__.

Although this guide will describe how to use it for central
authentication, LDAP is good for anything that involves a large number
of access requests to a mostly-read, attribute-based (name:value)
backend. Examples include an address book, a list of email addresses,
and a mail server's configuration.

Installation
------------

Install the OpenLDAP server daemon and the traditional LDAP management
utilities. These are found in packages slapd and ldap-utils
respectively.

The installation of slapd will create a working configuration. In
particular, it will create a database instance that you can use to store
your data. However, the suffix (or base DN) of this instance will be
determined from the domain name of the localhost. If you want something
different, edit ``/etc/hosts`` and replace the domain name with one that
will give you the suffix you desire. For instance, if you want a suffix
of *dc=example,dc=com* then your file would have a line similar to this:

::

    127.0.1.1       hostname.example.com    hostname

You can revert the change after package installation.

    **Note**

    This guide will use a database suffix of *dc=example,dc=com*.

Proceed with the install:

::

Since Ubuntu 8.10 slapd is designed to be configured within slapd itself
by dedicating a separate DIT for that purpose. This allows one to
dynamically configure slapd without the need to restart the service.
This configuration database consists of a collection of text-based LDIF
files located under ``/etc/ldap/slapd.d``. This way of working is known
by several names: the slapd-config method, the RTC method (Real Time
Configuration), or the cn=config method. You can still use the
traditional flat-file method (slapd.conf) but it's not recommended; the
functionality will be eventually phased out.

    **Note**

    Ubuntu now uses the *slapd-config* method for slapd configuration
    and this guide reflects that.

During the install you were prompted to define administrative
credentials. These are LDAP-based credentials for the *rootDN* of your
database instance. By default, this user's DN is
*cn=admin,dc=example,dc=com*. Also by default, there is no
administrative account created for the slapd-config database and you
will therefore need to authenticate externally to LDAP in order to
access it. We will see how to do this later on.

Some classical schemas (cosine, nis, inetorgperson) come built-in with
slapd nowadays. There is also an included "core" schema, a pre-requisite
for any schema to work.

Post-install Inspection
-----------------------

The installation process set up 2 DITs. One for slapd-config and one for
your own data (dc=example,dc=com). Let's take a look.

-  This is what the slapd-config database/DIT looks like. Recall that
   this database is LDIF-based and lives under ``/etc/ldap/slapd.d``:

   ::

       **Note**

       Do not edit the slapd-config database directly. Make changes via
       the LDAP protocol (utilities).

-  This is what the slapd-config DIT looks like via the LDAP protocol:

   ::


   Explanation of entries:

   -  *cn=config*: global settings

   -  *cn=module{0},cn=config*: a dynamically loaded module

   -  *cn=schema,cn=config*: contains hard-coded system-level schema

   -  *cn={0}core,cn=schema,cn=config*: the hard-coded core schema

   -  *cn={1}cosine,cn=schema,cn=config*: the cosine schema

   -  *cn={2}nis,cn=schema,cn=config*: the nis schema

   -  *cn={3}inetorgperson,cn=schema,cn=config*: the inetorgperson
      schema

   -  *olcBackend={0}hdb,cn=config*: the 'hdb' backend storage type

   -  *olcDatabase={-1}frontend,cn=config*: frontend database, default
      settings for other databases

   -  *olcDatabase={0}config,cn=config*: slapd configuration database
      (cn=config)

   -  *olcDatabase={1}hdb,cn=config*: your database instance
      (dc=examle,dc=com)

-  This is what the dc=example,dc=com DIT looks like:

   ::


   Explanation of entries:

   -  *dc=example,dc=com*: base of the DIT

   -  *cn=admin,dc=example,dc=com*: administrator (rootDN) for this DIT
      (set up during package install)

Modifying/Populating your Database
----------------------------------

Let's introduce some content to our database. We will add the following:

-  a node called *People* (to store users)

-  a node called *Groups* (to store groups)

-  a group called *miners*

-  a user called *john*

Create the following LDIF file and call it ``add_content.ldif``:

::

    dn: ou=People,dc=example,dc=com
    objectClass: organizationalUnit
    ou: People

    dn: ou=Groups,dc=example,dc=com
    objectClass: organizationalUnit
    ou: Groups

    dn: cn=miners,ou=Groups,dc=example,dc=com
    objectClass: posixGroup
    cn: miners
    gidNumber: 5000

    dn: uid=john,ou=People,dc=example,dc=com
    objectClass: inetOrgPerson
    objectClass: posixAccount
    objectClass: shadowAccount
    uid: john
    sn: Doe
    givenName: John
    cn: John Doe
    displayName: John Doe
    uidNumber: 10000
    gidNumber: 5000
    userPassword: johnldap
    gecos: John Doe
    loginShell: /bin/bash
    homeDirectory: /home/john

    **Note**

    It's important that uid and gid values in your directory do not
    collide with local values. Use high number ranges, such as starting
    at 5000. By setting the uid and gid values in ldap high, you also
    allow for easier control of what can be done with a local user vs a
    ldap one. More on that later.

Add the content:

::


We can check that the information has been correctly added with the
ldapsearch utility:

::


Explanation of switches:

-  *-x:* "simple" binding; will not use the default SASL method

-  *-LLL:* disable printing extraneous information

-  *uid=john:* a "filter" to find the john user

-  *cn gidNumber:* requests certain attributes to be displayed (the
   default is to show all attributes)

Modifying the slapd Configuration Database
------------------------------------------

The slapd-config DIT can also be queried and modified. Here are a few
examples.

-  Use ldapmodify to add an "Index" (DbIndex attribute) to your
   {1}hdb,cn=config database (dc=example,dc=com). Create a file, call it
   ``uid_index.ldif``, with the following contents:

   ::

       dn: olcDatabase={1}hdb,cn=config
       add: olcDbIndex
       olcDbIndex: uid eq,pres,sub

   Then issue the command:

   ::


   You can confirm the change in this way:

   ::


-  Let's add a schema. It will first need to be converted to LDIF
   format. You can find unconverted schemas in addition to converted
   ones in the ``/etc/ldap/schema`` directory.

       **Note**

       -  It is not trivial to remove a schema from the slapd-config
          database. Practice adding schemas on a test system.

       -  Before adding any schema, you should check which schemas are
          already installed (shown is a default, out-of-the-box output):

          ::


   In the following example we'll add the CORBA schema.

   Create the conversion configuration file ``schema_convert.conf``
   containing the following lines:

   ::

       include /etc/ldap/schema/core.schema
       include /etc/ldap/schema/collective.schema
       include /etc/ldap/schema/corba.schema
       include /etc/ldap/schema/cosine.schema
       include /etc/ldap/schema/duaconf.schema
       include /etc/ldap/schema/dyngroup.schema
       include /etc/ldap/schema/inetorgperson.schema
       include /etc/ldap/schema/java.schema
       include /etc/ldap/schema/misc.schema
       include /etc/ldap/schema/nis.schema
       include /etc/ldap/schema/openldap.schema
       include /etc/ldap/schema/ppolicy.schema
       include /etc/ldap/schema/ldapns.schema
       include /etc/ldap/schema/pmi.schema

   Create the output directory ``ldif_output``.

   Determine the index of the schema:

   ::


       **Note**

       When slapd ingests objects with the same parent DN it will create
       an *index* for that object. An index is contained within braces:
       {X}.

   Use slapcat to perform the conversion:

   ::

   The converted schema is now in ``cn=corba.ldif``

   Edit ``cn=corba.ldif`` to arrive at the following attributes:

   ::

       dn: cn=corba,cn=schema,cn=config
       ...
       cn: corba

   Also remove the following lines from the bottom:

   ::

       structuralObjectClass: olcSchemaConfig
       entryUUID: 52109a02-66ab-1030-8be2-bbf166230478
       creatorsName: cn=config
       createTimestamp: 20110829165435Z
       entryCSN: 20110829165435.935248Z#000000#000#000000
       modifiersName: cn=config
       modifyTimestamp: 20110829165435Z

   Your attribute values will vary.

   Finally, use ldapadd to add the new schema to the slapd-config DIT:

   ::


   Confirm currently loaded schemas:

   ::


    **Note**

    For external applications and clients to authenticate using LDAP
    they will each need to be specifically configured to do so. Refer to
    the appropriate client-side documentation for details.

Logging
-------

Activity logging for slapd is indispensible when implementing an
OpenLDAP-based solution yet it must be manually enabled after software
installation. Otherwise, only rudimentary messages will appear in the
logs. Logging, like any other slapd configuration, is enabled via the
slapd-config database.

OpenLDAP comes with multiple logging subsystems (levels) with each one
containing the lower one (additive). A good level to try is *stats*. The
`slapd-config <http://manpages.ubuntu.com/manpages/en/man5/slapd-config.5.html>`__
man page has more to say on the different subsystems.

Create the file ``logging.ldif`` with the following contents:

::

    dn: cn=config
    changetype: modify
    add: olcLogLevel
    olcLogLevel: stats

Implement the change:

::

This will produce a significant amount of logging and you will want to
throttle back to a less verbose level once your system is in production.
While in this verbose mode your host's syslog engine (rsyslog) may have
a hard time keeping up and may drop messages:

::

    rsyslogd-2177: imuxsock lost 228 messages from pid 2547 due to rate-limiting

You may consider a change to rsyslog's configuration. In
``/etc/rsyslog.conf``, put:

::

    # Disable rate limiting
    # (default is 200 messages in 5 seconds; below we make the 5 become 0)
    $SystemLogRateLimitInterval 0

And then restart the rsyslog daemon:

::

Replication
-----------

The LDAP service becomes increasingly important as more networked
systems begin to depend on it. In such an environment, it is standard
practice to build redundancy (high availability) into LDAP to prevent
havoc should the LDAP server become unresponsive. This is done through
*LDAP replication*.

Replication is achieved via the *Syncrepl* engine. This allows changes
to be synchronized using a *Consumer* - *Provider* model. The specific
kind of replication we will implement in this guide is a combination of
the following modes: *refreshAndPersist* and *delta-syncrepl*. This has
the Provider push changed entries to the Consumer as soon as they're
made but, in addition, only actual changes will be sent, not entire
entries.

Provider Configuration
~~~~~~~~~~~~~~~~~~~~~~

Begin by configuring the *Provider*.

Create an LDIF file with the following contents and name it
``provider_sync.ldif``:

::

    # Add indexes to the frontend db.
    dn: olcDatabase={1}hdb,cn=config
    changetype: modify
    add: olcDbIndex
    olcDbIndex: entryCSN eq
    -
    add: olcDbIndex
    olcDbIndex: entryUUID eq

    #Load the syncprov and accesslog modules.
    dn: cn=module{0},cn=config
    changetype: modify
    add: olcModuleLoad
    olcModuleLoad: syncprov
    -
    add: olcModuleLoad
    olcModuleLoad: accesslog

    # Accesslog database definitions
    dn: olcDatabase={2}hdb,cn=config
    objectClass: olcDatabaseConfig
    objectClass: olcHdbConfig
    olcDatabase: {2}hdb
    olcDbDirectory: /var/lib/ldap/accesslog
    olcSuffix: cn=accesslog
    olcRootDN: cn=admin,dc=example,dc=com
    olcDbIndex: default eq
    olcDbIndex: entryCSN,objectClass,reqEnd,reqResult,reqStart

    # Accesslog db syncprov.
    dn: olcOverlay=syncprov,olcDatabase={2}hdb,cn=config
    changetype: add
    objectClass: olcOverlayConfig
    objectClass: olcSyncProvConfig
    olcOverlay: syncprov
    olcSpNoPresent: TRUE
    olcSpReloadHint: TRUE

    # syncrepl Provider for primary db
    dn: olcOverlay=syncprov,olcDatabase={1}hdb,cn=config
    changetype: add
    objectClass: olcOverlayConfig
    objectClass: olcSyncProvConfig
    olcOverlay: syncprov
    olcSpNoPresent: TRUE

    # accesslog overlay definitions for primary db
    dn: olcOverlay=accesslog,olcDatabase={1}hdb,cn=config
    objectClass: olcOverlayConfig
    objectClass: olcAccessLogConfig
    olcOverlay: accesslog
    olcAccessLogDB: cn=accesslog
    olcAccessLogOps: writes
    olcAccessLogSuccess: TRUE
    # scan the accesslog DB every day, and purge entries older than 7 days
    olcAccessLogPurge: 07+00:00 01+00:00

Change the rootDN in the LDIF file to match the one you have for your
directory.

The apparmor profile for slapd will need to be adjusted for the
accesslog database location. Edit
``/etc/apparmor.d/local/usr.sbin.slapd`` by adding the following:

::

    /var/lib/ldap/accesslog/ r,
    /var/lib/ldap/accesslog/** rwk,

Create a directory, set up a databse config file, and reload the
apparmor profile:

::



Add the new content and, due to the apparmor change, restart the daemon:

::


The Provider is now configured.

Consumer Configuration
~~~~~~~~~~~~~~~~~~~~~~

And now configure the *Consumer*.

Install the software by going through ?. Make sure the slapd-config
databse is identical to the Provider's. In particular, make sure schemas
and the databse suffix are the same.

Create an LDIF file with the following contents and name it
``consumer_sync.ldif``:

::

    dn: cn=module{0},cn=config
    changetype: modify
    add: olcModuleLoad
    olcModuleLoad: syncprov

    dn: olcDatabase={1}hdb,cn=config
    changetype: modify
    add: olcDbIndex
    olcDbIndex: entryUUID eq
    -
    add: olcSyncRepl
    olcSyncRepl: rid=0 provider=ldap://ldap01.example.com bindmethod=simple binddn="cn=admin,dc=example,dc=com" 
     credentials=secret searchbase="dc=example,dc=com" logbase="cn=accesslog" 
     logfilter="(&(objectClass=auditWriteObject)(reqResult=0))" schemachecking=on 
     type=refreshAndPersist retry="60 +" syncdata=accesslog
    -
    add: olcUpdateRef
    olcUpdateRef: ldap://ldap01.example.com

Ensure the following attributes have the correct values:

-  *provider* (Provider server's hostname -- ldap01.example.com in this
   example -- or IP address)

-  *binddn* (the admin DN you're using)

-  *credentials* (the admin DN password you're using)

-  *searchbase* (the database suffix you're using)

-  *olcUpdateRef* (Provider server's hostname or IP address)

-  *rid* (Replica ID, an unique 3-digit that identifies the replica.
   Each consumer should have at least one rid)

Add the new content:

::

You're done. The two databases (suffix: dc=example,dc=com) should now be
synchronizing.

Testing
~~~~~~~

Once replication starts, you can monitor it by running

::


on both the provider and the consumer. Once the output
(``20120201193408.178454Z#000000#000#000000`` in the above example) for
both machines match, you have replication. Every time a change is done
in the provider, this value will change and so should the one in the
consumer(s).

If your connection is slow and/or your ldap database large, it might
take a while for the consumer's *contextCSN* match the provider's. But,
you will know it is progressing since the consumer's *contextCSN* will
be steadly increasing.

If the consumer's *contextCSN* is missing or does not match the
provider, you should stop and figure out the issue before continuing.
Try checking the slapd (syslog) and the auth log files in the provider
to see if the consumer's authentication requests were successful or its
requests to retrieve data (they look like a lot of ldapsearch
statements) return no errors.

To test if it worked simply query, on the Consumer, the DNs in the
database:

::

You should see the user 'john' and the group 'miners' as well as the
nodes 'People' and 'Groups'.

Access Control
--------------

The management of what type of access (read, write, etc) users should be
granted to resources is known as *access control*. The configuration
directives involved are called *access control lists* or ACL.

When we installed the slapd package various ACL were set up
automatically. We will look at a few important consequences of those
defaults and, in so doing, we'll get an idea of how ACLs work and how
they're configured.

To get the effective ACL for an LDAP query we need to look at the ACL
entries of the database being queried as well as those of the special
frontend database instance. The ACLs belonging to the latter act as
defaults in case those of the former do not match. The frontend database
is the second to be consulted and the ACL to be applied is the first to
match ("first match wins") among these 2 ACL sources. The following
commands will give, respectively, the ACLs of the hdb database
("dc=example,dc=com") and those of the frontend database:

::


    **Note**

    The rootDN always has full rights to it's database. Including it in
    an ACL does provide an explicit configuration but it also causes
    slapd to incur a performance penalty.

::


The very first ACL is crucial:

::

    olcAccess: {0}to attrs=userPassword,shadowLastChange by self write by anonymous
                  auth by dn="cn=admin,dc=example,dc=com" write by * none

This can be represented differently for easier digestion:

::

    to attrs=userPassword
        by self write
        by anonymous auth
        by dn="cn=admin,dc=example,dc=com" write
        by * none

    to attrs=shadowLastChange
        by self write
        by anonymous auth
        by dn="cn=admin,dc=example,dc=com" write
        by * none

This compound ACL (there are 2) enforces the following:

-  Anonymous 'auth' access is provided to the *userPassword* attribute
   for the initial connection to occur. Perhaps counter-intuitively, 'by
   anonymous auth' is needed even when anonymous access to the DIT is
   unwanted. Once the remote end is connected, howerver, authentication
   can occur (see next point).

-  Authentication can happen because all users have 'read' (due to 'by
   self write') access to the *userPassword* attribute.

-  The *userPassword* attribute is otherwise unaccessible by all other
   users, with the exception of the rootDN, who has complete access to
   it.

-  In order for users to change their own password, using ``passwd`` or
   other utilities, the *shadowLastChange* attribute needs to be
   accessible once a user has authenticated.

This DIT can be searched anonymously because of 'by \* read' in this
ACL:

::

    to *
        by self write
        by dn="cn=admin,dc=example,dc=com" write
        by * read

If this is unwanted then you need to change the ACLs. To force
authentication during a bind request you can alternatively (or in
combination with the modified ACL) use the 'olcRequire: authc'
directive.

As previously mentioned, there is no administrative account created for
the slapd-config database. There is, however, a SASL identity that is
granted full access to it. It represents the localhost's superuser
(root/sudo). Here it is:

::

    dn.exact=gidNumber=0+uidNumber=0,cn=peercred,cn=external,cn=auth 

The following command will display the ACLs of the slapd-config
database:

::


Since this is a SASL identity we need to use a SASL *mechanism* when
invoking the LDAP utility in question and we have seen it plenty of
times in this guide. It is the EXTERNAL mechanism. See the previous
command for an example. Note that:

You must use *sudo* to become the root identity in order for the ACL to
match.

The EXTERNAL mechanism works via *IPC* (UNIX domain sockets). This means
you must use the *ldapi* URI format.

A succinct way to get all the ACLs is like this:

::

There is much to say on the topic of access control. See the man page
for
`slapd.access <http://manpages.ubuntu.com/manpages/en/man5/slapd.access.5.html>`__.

TLS
---

When authenticating to an OpenLDAP server it is best to do so using an
encrypted session. This can be accomplished using Transport Layer
Security (TLS).

Here, we will be our own *Certificate Authority* and then create and
sign our LDAP server certificate as that CA. Since slapd is compiled
using the gnutls library, we will use the certtool utility to complete
these tasks.

Install the gnutls-bin and ssl-cert packages:

::

Create a private key for the Certificate Authority:

::

Create the template/file ``/etc/ssl/ca.info`` to define the CA:

::

    cn = Example Company
    ca
    cert_signing_key

Create the self-signed CA certificate:

::

Make a private key for the server:

::

    **Note**

    Replace *ldap01* in the filename with your server's hostname. Naming
    the certificate and key for the host and service that will be using
    them will help keep things clear.

Create the ``/etc/ssl/ldap01.info`` info file containing:

::

    organization = Example Company
    cn = ldap01.example.com
    tls_www_server
    encryption_key
    signing_key
    expiration_days = 3650

The above certificate is good for 10 years. Adjust accordingly.

Create the server's certificate:

::

Create the file ``certinfo.ldif`` with the following contents (adjust
accordingly, our example assumes we created certs using
https://www.cacert.org):

::

    dn: cn=config
    add: olcTLSCACertificateFile
    olcTLSCACertificateFile: /etc/ssl/certs/cacert.pem
    -
    add: olcTLSCertificateFile
    olcTLSCertificateFile: /etc/ssl/certs/ldap01_slapd_cert.pem
    -
    add: olcTLSCertificateKeyFile
    olcTLSCertificateKeyFile: /etc/ssl/private/ldap01_slapd_key.pem

Use the ldapmodify command to tell slapd about our TLS work via the
slapd-config database:

::

Contratry to popular belief, you do not need *ldaps://* in
``/etc/default/slapd`` in order to use encryption. You should have just:

::

    SLAPD_SERVICES="ldap:/// ldapi:///"

    **Note**

    LDAP over TLS/SSL (ldaps://) is deprecated in favour of *StartTLS*.
    The latter refers to an existing LDAP session (listening on TCP port
    389) becoming protected by TLS/SSL whereas LDAPS, like HTTPS, is a
    distinct encrypted-from-the-start protocol that operates over TCP
    port 636.

Tighten up ownership and permissions:

::




Restart OpenLDAP:

::

Check your host's logs (/var/log/syslog) to see if the server has
started properly.

Replication and TLS
-------------------

If you have set up replication between servers, it is common practice to
encrypt (StartTLS) the replication traffic to prevent evesdropping. This
is distinct from using encryption with authentication as we did above.
In this section we will build on that TLS-authentication work.

The assumption here is that you have set up replication between Provider
and Consumer according to ? and have configured TLS for authentication
on the Provider by following ?.

As previously stated, the objective (for us) with replication is high
availablity for the LDAP service. Since we have TLS for authentication
on the Provider we will require the same on the Consumer. In addition to
this, however, we want to encrypt replication traffic. What remains to
be done is to create a key and certificate for the Consumer and then
configure accordingly. We will generate the key/certificate on the
Provider, to avoid having to create another CA certificate, and then
transfer the necessary material over to the Consumer.

On the Provider,

Create a holding directory (which will be used for the eventual
transfer) and then the Consumer's private key:

::



Create an info file, ``ldap02.info``, for the Consumer server, adjusting
it's values accordingly:

::

    organization = Example Company
    cn = ldap02.example.com
    tls_www_server
    encryption_key
    signing_key
    expiration_days = 3650

Create the Consumer's certificate:

::

Get a copy of the CA certificate:

::

We're done. Now transfer the ``ldap02-ssl`` directory to the Consumer.
Here we use scp (adjust accordingly):

::


On the Consumer,

Configure TLS authentication:

::







Create the file ``/etc/ssl/certinfo.ldif`` with the following contents
(adjust accordingly):

::

    dn: cn=config
    add: olcTLSCACertificateFile
    olcTLSCACertificateFile: /etc/ssl/certs/cacert.pem
    -
    add: olcTLSCertificateFile
    olcTLSCertificateFile: /etc/ssl/certs/ldap02_slapd_cert.pem
    -
    add: olcTLSCertificateKeyFile
    olcTLSCertificateKeyFile: /etc/ssl/private/ldap02_slapd_key.pem

Configure the slapd-config database:

::

Configure ``/etc/default/slapd`` as on the Provider (SLAPD\_SERVICES).

On the Consumer,

Configure TLS for Consumer-side replication. Modify the existing
*olcSyncrepl* attribute by tacking on some TLS options. In so doing, we
will see, for the first time, how to change an attribute's value(s).

Create the file ``consumer_sync_tls.ldif`` with the following contents:

::

    dn: olcDatabase={1}hdb,cn=config
    replace: olcSyncRepl
    olcSyncRepl: rid=0 provider=ldap://ldap01.example.com bindmethod=simple
     binddn="cn=admin,dc=example,dc=com" credentials=secret searchbase="dc=example,dc=com"
     logbase="cn=accesslog" logfilter="(&(objectClass=auditWriteObject)(reqResult=0))"
     schemachecking=on type=refreshAndPersist retry="60 +" syncdata=accesslog
     

The extra options specify, respectively, that the consumer must use
StartTLS and that the CA certificate is required to verify the
Provider's identity. Also note the LDIF syntax for changing the values
of an attribute ('replace').

Implement these changes:

::

And restart slapd:

::

On the Provider,

Check to see that a TLS session has been established. In
``/var/log/syslog``, providing you have 'conns'-level logging set up,
you should see messages similar to:

::

    slapd[3620]: conn=1047 fd=20 ACCEPT from IP=10.153.107.229:57922 (IP=0.0.0.0:389)
    slapd[3620]: conn=1047 op=0 EXT oid=1.3.6.1.4.1.1466.20037
    slapd[3620]: conn=1047 op=0 STARTTLS
    slapd[3620]: conn=1047 op=0 RESULT oid= err=0 text=
    slapd[3620]: conn=1047 fd=20 TLS established tls_ssf=128 ssf=128
    slapd[3620]: conn=1047 op=1 BIND dn="cn=admin,dc=example,dc=com" method=128
    slapd[3620]: conn=1047 op=1 BIND dn="cn=admin,dc=example,dc=com" mech=SIMPLE ssf=0
    slapd[3620]: conn=1047 op=1 RESULT tag=97 err=0 text

LDAP Authentication
-------------------

Once you have a working LDAP server, you will need to install libraries
on the client that will know how and when to contact it. On Ubuntu, this
has been traditionally accomplished by installing the libnss-ldap
package. This package will bring in other tools that will assist you in
the configuration step. Install this package now:

::

You will be prompted for details of your LDAP server. If you make a
mistake you can try again using:

::

The results of the dialog can be seen in ``/etc/ldap.conf``. If your
server requires options not covered in the menu edit this file
accordingly.

Now configure the LDAP profile for NSS:

::

Configure the system to use LDAP for authentication:

::

From the menu, choose LDAP and any other authentication mechanisms you
need.

You should now be able to log in using LDAP-based credentials.

LDAP clients will need to refer to multiple servers if replication is in
use. In ``/etc/ldap.conf`` you would have something like:

::

    uri ldap://ldap01.example.com ldap://ldap02.example.com

The request will time out and the Consumer (ldap02) will attempt to be
reached if the Provider (ldap01) becomes unresponsive.

If you are going to use LDAP to store Samba users you will need to
configure the Samba server to authenticate using LDAP. See ? for
details.

    **Note**

    An alternative to the libnss-ldap package is the libnss-ldapd
    package. This, however, will bring in the nscd package which is
    problably not wanted. Simply remove it afterwards.

User and Group Management
-------------------------

The ldap-utils package comes with enough utilities to manage the
directory but the long string of options needed can make them a burden
to use. The ldapscripts package contains wrapper scripts to these
utilities that some people find easier to use.

Install the package:

::

Then edit the file ``/etc/ldapscripts/ldapscripts.conf`` to arrive at
something similar to the following:

::

    SERVER=localhost
    BINDDN='cn=admin,dc=example,dc=com'
    BINDPWDFILE="/etc/ldapscripts/ldapscripts.passwd"
    SUFFIX='dc=example,dc=com'
    GSUFFIX='ou=Groups'
    USUFFIX='ou=People'
    MSUFFIX='ou=Computers'
    GIDSTART=10000
    UIDSTART=10000
    MIDSTART=10000

Now, create the ``ldapscripts.passwd`` file to allow rootDN access to
the directory:

::


    **Note**

    Replace “secret” with the actual password for your database's rootDN
    user.

The scripts are now ready to help manage your directory. Here are some
examples of how to use them:

-  Create a new user:

   ::

   This will create a user with uid *george* and set the user's primary
   group (gid) to *example*

-  Change a user's password:

   ::




-  Delete a user:

   ::

-  Add a group:

   ::

-  Delete a group:

   ::

-  Add a user to a group:

   ::

   You should now see a *memberUid* attribute for the *qa* group with a
   value of *george*.

-  Remove a user from a group:

   ::

   The *memberUid* attribute should now be removed from the *qa* group.

-  The ldapmodifyuser script allows you to add, remove, or replace a
   user's attributes. The script uses the same syntax as the ldapmodify
   utility. For example:

   ::



   The user's *gecos* should now be “George Carlin”.

-  A nice feature of ldapscripts is the template system. Templates allow
   you to customize the attributes of user, group, and machine objects.
   For example, to enable the *user* template edit
   ``/etc/ldapscripts/ldapscripts.conf`` changing:

   ::

       UTEMPLATE="/etc/ldapscripts/ldapadduser.template"

   There are *sample* templates in the ``/etc/ldapscripts`` directory.
   Copy or rename the ``ldapadduser.template.sample`` file to
   ``/etc/ldapscripts/ldapadduser.template``:

   ::

   Edit the new template to add the desired attributes. The following
   will create new users with an objectClass of inetOrgPerson:

   ::

       dn: uid=<user>,<usuffix>,<suffix>
       objectClass: inetOrgPerson
       objectClass: posixAccount
       cn: <user>
       sn: <ask>
       uid: <user>
       uidNumber: <uid>
       gidNumber: <gid>
       homeDirectory: <home>
       loginShell: <shell>
       gecos: <user>
       description: User account
       title: Employee

   Notice the *<ask>* option used for the *sn* attribute. This will make
   ldapadduser prompt you for it's value.

There are utilities in the package that were not covered here. Here is a
complete list:

::





















Backup and Restore
------------------

Now we have ldap running just the way we want, it is time to ensure we
can save all of our work and restore it as needed.

What we need is a way to backup the ldap database(s), specifically the
backend (cn=config) and frontend (dc=example,dc=com). If we are going to
backup those databases into, say, ``/export/backup``, we could use
slapcat as shown in the following script, called
``/usr/local/bin/ldapbackup``:

::

    #!/bin/bash

    BACKUP_PATH=/export/backup
    SLAPCAT=/usr/sbin/slapcat

    nice ${SLAPCAT} -n 0 > ${BACKUP_PATH}/config.ldif
    nice ${SLAPCAT} -n 1 > ${BACKUP_PATH}/example.com.ldif
    nice ${SLAPCAT} -n 2 > ${BACKUP_PATH}/access.ldif
    chmod 640 ${BACKUP_PATH}/*.ldif

    **Note**

    These files are uncompressed text files containing everything in
    your ldap databases including the tree layout, usernames, and every
    password. So, you might want to consider making ``/export/backup``
    an encrypted partition and even having the script encrypt those
    files as it creates them. Ideally you should do both, but that
    depends on your security requirements.

Then, it is just a matter of having a cron script to run this program as
often as we feel comfortable with. For many, once a day suffices. For
others, more often is required. Here is an example of a cron script
called ``/etc/cron.d/ldapbackup`` that is run every night at 22:45h:

::

    MAILTO=backup-emails@domain.com
    45 22 * * *  root    /usr/local/bin/ldapbackup

Now the files are created, they should be copied to a backup server.

Assuming we did a fresh reinstall of ldap, the restore process could be
something like this:

::








Resources
---------

-  The primary resource is the upstream documentation:
   `www.openldap.org <http://www.openldap.org/>`__

-  There are many man pages that come with the slapd package. Here are
   some important ones, especially considering the material presented in
   this guide:

   ::




-  Other man pages:

   ::


-  Zytrax's `LDAP for Rocket
   Scientists <http://www.zytrax.com/books/ldap/>`__; a less pedantic
   but comprehensive treatment of LDAP

-  A Ubuntu community `OpenLDAP
   wiki <https://help.ubuntu.com/community/OpenLDAPServer>`__ page has a
   collection of notes

-  O'Reilly's `LDAP System
   Administration <http://www.oreilly.com/catalog/ldapsa/>`__ (textbook;
   2003)

-  Packt's `Mastering
   OpenLDAP <http://www.packtpub.com/OpenLDAP-Developers-Server-Open-Source-Linux/book>`__
   (textbook; 2007)

Samba and LDAP
==============

This section covers the integration of Samba with LDAP. The Samba
server's role will be that of a "standalone" server and the LDAP
directory will provide the authentication layer in addition to
containing the user, group, and machine account information that Samba
requires in order to function (in any of it's 3 possible roles). The
pre-requisite is an OpenLDAP server configured with a directory that can
accept authentication requests. See ? for details on fulfilling this
requirement. Once this section is completed, you will need to decide
what specifically you want Samba to do for you and then configure it
accordingly.

Software Installation
---------------------

There are three packages needed when integrating Samba with LDAP: samba,
samba-doc, and smbldap-tools packages.

Strictly speaking, the smbldap-tools package isn't needed, but unless
you have some other way to manage the various Samba entities (users,
groups, computers) in an LDAP context then you should install it.

Install these packages now:

::

LDAP Configuration
------------------

We will now configure the LDAP server so that it can accomodate Samba
data. We will perform three tasks in this section:

Import a schema

Index some entries

Add objects

Samba schema
~~~~~~~~~~~~

In order for OpenLDAP to be used as a backend for Samba, logically, the
DIT will need to use attributes that can properly describe Samba data.
Such attributes can be obtained by introducing a Samba LDAP schema.
Let's do this now.

    **Note**

    For more information on schemas and their installation see ?.

The schema is found in the now-installed samba-doc package. It needs to
be unzipped and copied to the ``/etc/ldap/schema`` directory:

::


Have the configuration file ``schema_convert.conf`` that contains the
following lines:

::

    include /etc/ldap/schema/core.schema
    include /etc/ldap/schema/collective.schema
    include /etc/ldap/schema/corba.schema
    include /etc/ldap/schema/cosine.schema
    include /etc/ldap/schema/duaconf.schema
    include /etc/ldap/schema/dyngroup.schema
    include /etc/ldap/schema/inetorgperson.schema
    include /etc/ldap/schema/java.schema
    include /etc/ldap/schema/misc.schema
    include /etc/ldap/schema/nis.schema
    include /etc/ldap/schema/openldap.schema
    include /etc/ldap/schema/ppolicy.schema
    include /etc/ldap/schema/ldapns.schema
    include /etc/ldap/schema/pmi.schema
    include /etc/ldap/schema/samba.schema

Have the directory ``ldif_output`` hold output.

Determine the index of the schema:

::


Convert the schema to LDIF format:

::

Edit the generated ``cn=samba.ldif`` file by removing index information
to arrive at:

::

    dn: cn=samba,cn=schema,cn=config
    ...
    cn: samba

Remove the bottom lines:

::

    structuralObjectClass: olcSchemaConfig
    entryUUID: b53b75ca-083f-102d-9fff-2f64fd123c95
    creatorsName: cn=config
    createTimestamp: 20080827045234Z
    entryCSN: 20080827045234.341425Z#000000#000#000000
    modifiersName: cn=config
    modifyTimestamp: 20080827045234Z

Your attribute values will vary.

Add the new schema:

::

To query and view this new schema:

::

Samba indices
~~~~~~~~~~~~~

Now that slapd knows about the Samba attributes, we can set up some
indices based on them. Indexing entries is a way to improve performance
when a client performs a filtered search on the DIT.

Create the file ``samba_indices.ldif`` with the following contents:

::

    dn: olcDatabase={1}hdb,cn=config
    changetype: modify
    add: olcDbIndex
    olcDbIndex: uidNumber eq
    olcDbIndex: gidNumber eq
    olcDbIndex: loginShell eq
    olcDbIndex: uid eq,pres,sub
    olcDbIndex: memberUid eq,pres,sub
    olcDbIndex: uniqueMember eq,pres
    olcDbIndex: sambaSID eq
    olcDbIndex: sambaPrimaryGroupSID eq
    olcDbIndex: sambaGroupType eq
    olcDbIndex: sambaSIDList eq
    olcDbIndex: sambaDomainName eq
    olcDbIndex: default sub

Using the ldapmodify utility load the new indices:

::

If all went well you should see the new indices using ldapsearch:

::

Adding Samba LDAP objects
~~~~~~~~~~~~~~~~~~~~~~~~~

Next, configure the smbldap-tools package to match your environment. The
package is supposed to come with a configuration helper script
(smbldap-config.pl, formerly configure.pl) that will ask questions about
the needed options but there is a
`bug <https://bugs.launchpad.net/serverguide/+bug/997172>`__ whereby it
is not installed (but found in the source code; 'apt-get source
smbldap-tools').

To manually configure the package, you need to create and edit the files
``/etc/smbldap-tools/smbldap.conf`` and
``/etc/smbldap-tools/smbldap_bind.conf``.

The smbldap-populate script will then add the LDAP objects required for
Samba. It is a good idea to first make a backup of your DIT using
slapcat:

::

Once you have a backup proceed to populate your directory:

::

You can create a LDIF file containing the new Samba objects by executing
``sudo smbldap-populate -e samba.ldif``. This allows you to look over
the changes making sure everything is correct. If it is, rerun the
script without the '-e' switch. Alternatively, you can take the LDIF
file and import it's data per usual.

Your LDAP directory now has the necessary information to authenticate
Samba users.

Samba Configuration
-------------------

There are multiple ways to configure Samba. For details on some common
configurations see ?. To configure Samba to use LDAP, edit it's
configuration file ``/etc/samba/smb.conf`` commenting out the default
*passdb backend* parameter and adding some ldap-related ones:

::

    #   passdb backend = tdbsam

    # LDAP Settings
       passdb backend = ldapsam:ldap://hostname
       ldap suffix = dc=example,dc=com
       ldap user suffix = ou=People
       ldap group suffix = ou=Groups
       ldap machine suffix = ou=Computers
       ldap idmap suffix = ou=Idmap
       ldap admin dn = cn=admin,dc=example,dc=com
       ldap ssl = start tls
       ldap passwd sync = yes
    ...
       add machine script = sudo /usr/sbin/smbldap-useradd -t 0 -w "%u"

Change the values to match your environment.

Restart samba to enable the new settings:

::


Now inform Samba about the rootDN user's password (the one set during
the installation of the slapd package):

::

If you have existing LDAP users that you want to include in your new
LDAP-backed Samba they will, of course, also need to be given some of
the extra attributes. The smbpasswd utility can do this as well (your
host will need to be able to see (enumerate) those users via NSS;
install and configure either libnss-ldapd or libnss-ldap):

::

You will prompted to enter a password. It will be considered as the new
password for that user. Making it the same as before is reasonable.

To manage user, group, and machine accounts use the utilities provided
by the smbldap-tools package. Here are some examples:

-  To add a new user:

   ::

   The *-a* option adds the Samba attributes, and the *-P* option calls
   the smbldap-passwd utility after the user is created allowing you to
   enter a password for the user.

-  To remove a user:

   ::

   In the above command, use the *-r* option to remove the user's home
   directory.

-  To add a group:

   ::

   As for smbldap-useradd, the *-a* adds the Samba attributes.

-  To make an existing user a member of a group:

   ::

   The *-m* option can add more than one user at a time by listing them
   in comma-separated format.

-  To remove a user from a group:

   ::

-  To add a Samba machine account:

   ::

   Replace *username* with the name of the workstation. The *-t 0*
   option creates the machine account without a delay, while the *-w*
   option specifies the user as a machine account. Also, note the *add
   machine script* parameter in ``/etc/samba/smb.conf`` was changed to
   use smbldap-useradd.

There are utilities in the smbldap-tools package that were not covered
here. Here is a complete list:

::












Resources
---------

-  For more information on installing and configuring Samba see ? of
   this Ubuntu Server Guide.

-  There are multiple places where LDAP and Samba is documented in the
   upstream `Samba HOWTO
   Collection <http://samba.org/samba/docs/man/Samba-HOWTO-Collection/>`__.

-  Regarding the above, see specifically the `passdb
   section <http://samba.org/samba/docs/man/Samba-HOWTO-Collection/passdb.html>`__.

-  Although dated (2007), the `Linux Samba-OpenLDAP
   HOWTO <http://download.gna.org/smbldap-tools/docs/samba-ldap-howto/>`__
   contains valuable notes.

-  The main page of the `Samba Ubuntu community
   documentation <https://help.ubuntu.com/community/Samba#samba-ldap>`__
   has a plethora of links to articles that may prove useful.

Kerberos
========

Kerberos is a network authentication system based on the principal of a
trusted third party. The other two parties being the user and the
service the user wishes to authenticate to. Not all services and
applications can use Kerberos, but for those that can, it brings the
network environment one step closer to being Single Sign On (SSO).

This section covers installation and configuration of a Kerberos server,
and some example client configurations.

Overview
--------

If you are new to Kerberos there are a few terms that are good to
understand before setting up a Kerberos server. Most of the terms will
relate to things you may be familiar with in other environments:

-  *Principal:* any users, computers, and services provided by servers
   need to be defined as Kerberos Principals.

-  *Instances:* are used for service principals and special
   administrative principals.

-  *Realms:* the unique realm of control provided by the Kerberos
   installation. Think of it as the domain or group your hosts and users
   belong to. Convention dictates the realm should be in uppercase. By
   default, ubuntu will use the DNS domain converted to uppercase
   (EXAMPLE.COM) as the realm.

-  *Key Distribution Center:* (KDC) consist of three parts, a database
   of all principals, the authentication server, and the ticket granting
   server. For each realm there must be at least one KDC.

-  *Ticket Granting Ticket:* issued by the Authentication Server (AS),
   the Ticket Granting Ticket (TGT) is encrypted in the user's password
   which is known only to the user and the KDC.

-  *Ticket Granting Server:* (TGS) issues service tickets to clients
   upon request.

-  *Tickets:* confirm the identity of the two principals. One principal
   being a user and the other a service requested by the user. Tickets
   establish an encryption key used for secure communication during the
   authenticated session.

-  *Keytab Files:* are files extracted from the KDC principal database
   and contain the encryption key for a service or host.

To put the pieces together, a Realm has at least one KDC, preferably
more for redundancy, which contains a database of Principals. When a
user principal logs into a workstation that is configured for Kerberos
authentication, the KDC issues a Ticket Granting Ticket (TGT). If the
user supplied credentials match, the user is authenticated and can then
request tickets for Kerberized services from the Ticket Granting Server
(TGS). The service tickets allow the user to authenticate to the service
without entering another username and password.

Kerberos Server
---------------

Installation
~~~~~~~~~~~~

For this discussion, we will create a MIT Kerberos domain with the
following features (edit them to fit your needs):

-  *Realm:* EXAMPLE.COM

-  *Primary KDC:* kdc01.example.com (192.168.0.1)

-  *Secondary KDC:* kdc02.example.com (192.168.0.2)

-  *User principal:* steve

-  *Admin principal:* steve/admin

    **Note**

    It is *strongly* recommended that your network-authenticated users
    have their uid in a different range (say, starting at 5000) than
    that of your local users.

Before installing the Kerberos server a properly configured DNS server
is needed for your domain. Since the Kerberos Realm by convention
matches the domain name, this section uses the *EXAMPLE.COM* domain
configured in ? of the DNS documentation.

Also, Kerberos is a time sensitive protocol. So if the local system time
between a client machine and the server differs by more than five
minutes (by default), the workstation will not be able to authenticate.
To correct the problem all hosts should have their time synchronized
using the same *Network Time Protocol (NTP)* server. For details on
setting up NTP see ?.

The first step in creating a Kerberos Realm is to install the krb5-kdc
and krb5-admin-server packages. From a terminal enter:

::

You will be asked at the end of the install to supply the hostname for
the Kerberos and Admin servers, which may or may not be the same server,
for the realm.

    **Note**

    By default the realm is created from the KDC's domain name.

Next, create the new realm with the kdb5\_newrealm utility:

::

Configuration
~~~~~~~~~~~~~

The questions asked during installation are used to configure the
``/etc/krb5.conf`` file. If you need to adjust the Key Distribution
Center (KDC) settings simply edit the file and restart the krb5-kdc
daemon. If you need to reconfigure Kerberos from scratch, perhaps to
change the realm name, you can do so by typing

::

Once the KDC is properly running, an admin user -- the *admin principal*
-- is needed. It is recommended to use a different username from your
everyday username. Using the kadmin.local utility in a terminal prompt
enter:

::



In the above example *steve* is the *Principal*, */admin* is an
*Instance*, and *@EXAMPLE.COM* signifies the realm. The *"every day"*
Principal, a.k.a. the *user principal*, would be *steve@EXAMPLE.COM*,
and should have only normal user rights.

    **Note**

    Replace *EXAMPLE.COM* and *steve* with your Realm and admin
    username.

Next, the new admin user needs to have the appropriate Access Control
List (ACL) permissions. The permissions are configured in the
``/etc/krb5kdc/kadm5.acl`` file:

::

    steve/admin@EXAMPLE.COM        *

This entry grants *steve/admin* the ability to perform any operation on
all principals in the realm. You can configure principals with more
restrictive privileges, which is convenient if you need an admin
principal that junior staff can use in Kerberos clients. Please see the
*kadm5.acl* man page for details.

Now restart the krb5-admin-server for the new ACL to take affect:

::

The new user principal can be tested using the kinit utility:

::


After entering the password, use the klist utility to view information
about the Ticket Granting Ticket (TGT):

::


Where the cache filename ``krb5cc_1000`` is composed of the prefix
``krb5cc_`` and the user id (uid), which in this case is ``1000``. You
may need to add an entry into the ``/etc/hosts`` for the KDC so the
client can find the KDC. For example:

::

    192.168.0.1   kdc01.example.com       kdc01

Replacing *192.168.0.1* with the IP address of your KDC. This usually
happens when you have a Kerberos realm encompassing different networks
separated by routers.

The best way to allow clients to automatically determine the KDC for the
Realm is using DNS SRV records. Add the following to
``/etc/named/db.example.com``:

::

    _kerberos._udp.EXAMPLE.COM.     IN SRV 1  0 88  kdc01.example.com.
    _kerberos._tcp.EXAMPLE.COM.     IN SRV 1  0 88  kdc01.example.com.
    _kerberos._udp.EXAMPLE.COM.     IN SRV 10 0 88  kdc02.example.com. 
    _kerberos._tcp.EXAMPLE.COM.     IN SRV 10 0 88  kdc02.example.com. 
    _kerberos-adm._tcp.EXAMPLE.COM. IN SRV 1  0 749 kdc01.example.com.
    _kpasswd._udp.EXAMPLE.COM.      IN SRV 1  0 464 kdc01.example.com.

    **Note**

    Replace *EXAMPLE.COM*, *kdc01*, and *kdc02* with your domain name,
    primary KDC, and secondary KDC.

See ? for detailed instructions on setting up DNS.

Your new Kerberos Realm is now ready to authenticate clients.

Secondary KDC
-------------

Once you have one Key Distribution Center (KDC) on your network, it is
good practice to have a Secondary KDC in case the primary becomes
unavailable. Also, if you have Kerberos clients that are in different
networks (possibly separated by routers using NAT), it is wise to place
a secondary KDC in each of those networks.

First, install the packages, and when asked for the Kerberos and Admin
server names enter the name of the Primary KDC:

::

Once you have the packages installed, create the Secondary KDC's host
principal. From a terminal prompt, enter:

::

    **Note**

    After, issuing any kadmin commands you will be prompted for your
    *username/admin@EXAMPLE.COM* principal password.

Extract the *keytab* file:

::

There should now be a ``keytab.kdc02`` in the current directory, move
the file to ``/etc/krb5.keytab``:

::

    **Note**

    If the path to the ``keytab.kdc02`` file is different adjust
    accordingly.

Also, you can list the principals in a Keytab file, which can be useful
when troubleshooting, using the klist utility:

::

The -k option indicates the file is a keytab file.

Next, there needs to be a ``kpropd.acl`` file on each KDC that lists all
KDCs for the Realm. For example, on both primary and secondary KDC,
create ``/etc/krb5kdc/kpropd.acl``:

::

    host/kdc01.example.com@EXAMPLE.COM
    host/kdc02.example.com@EXAMPLE.COM

Create an empty database on the *Secondary KDC*:

::

Now start the kpropd daemon, which listens for connections from the
kprop utility. kprop is used to transfer dump files:

::

From a terminal on the *Primary KDC*, create a dump file of the
principal database:

::

Extract the Primary KDC's *keytab* file and copy it to
``/etc/krb5.keytab``:

::


    **Note**

    Make sure there is a *host* for *kdc01.example.com* before
    extracting the Keytab.

Using the kprop utility push the database to the Secondary KDC:

::

    **Note**

    There should be a *SUCCEEDED* message if the propagation worked. If
    there is an error message check ``/var/log/syslog`` on the secondary
    KDC for more information.

You may also want to create a cron job to periodically update the
database on the Secondary KDC. For example, the following will push the
database every hour (note the long line has been split to fit the format
of this document):

::

    # m h  dom mon dow   command
    0 * * * * /usr/sbin/kdb5_util dump /var/lib/krb5kdc/dump && 
    /usr/sbin/kprop -r EXAMPLE.COM -f /var/lib/krb5kdc/dump kdc02.example.com

Back on the *Secondary KDC*, create a *stash* file to hold the Kerberos
master key:

::

Finally, start the krb5-kdc daemon on the Secondary KDC:

::

The *Secondary KDC* should now be able to issue tickets for the Realm.
You can test this by stopping the krb5-kdc daemon on the Primary KDC,
then by using kinit to request a ticket. If all goes well you should
receive a ticket from the Secondary KDC. Otherwise, check
``/var/log/syslog`` and ``/var/log/auth.log`` in the Secondary KDC.

Kerberos Linux Client
---------------------

This section covers configuring a Linux system as a Kerberos client.
This will allow access to any kerberized services once a user has
successfully logged into the system.

Installation
~~~~~~~~~~~~

In order to authenticate to a Kerberos Realm, the krb5-user and
libpam-krb5 packages are needed, along with a few others that are not
strictly necessary but make life easier. To install the packages enter
the following in a terminal prompt:

::

The auth-client-config package allows simple configuration of PAM for
authentication from multiple sources, and the libpam-ccreds will cache
authentication credentials allowing you to login in case the Key
Distribution Center (KDC) is unavailable. This package is also useful
for laptops that may authenticate using Kerberos while on the corporate
network, but will need to be accessed off the network as well.

Configuration
~~~~~~~~~~~~~

To configure the client in a terminal enter:

::

You will then be prompted to enter the name of the Kerberos Realm. Also,
if you don't have DNS configured with Kerberos *SRV* records, the menu
will prompt you for the hostname of the Key Distribution Center (KDC)
and Realm Administration server.

The dpkg-reconfigure adds entries to the ``/etc/krb5.conf`` file for
your Realm. You should have entries similar to the following:

::

    [libdefaults]
            default_realm = EXAMPLE.COM
    ...
    [realms]
            EXAMPLE.COM = {
                    kdc = 192.168.0.1
                    admin_server = 192.168.0.1
            }

    **Note**

    If you set the uid of each of your network-authenticated users to
    start at 5000, as suggested in ?, you can then tell pam to only try
    to authenticate using Kerberos users with uid > 5000:

    ::

    This will avoid being asked for the (non-existent) Kerberos password
    of a locally authenticated user when changing its password using
    ``passwd``.

You can test the configuration by requesting a ticket using the kinit
utility. For example:

::


When a ticket has been granted, the details can be viewed using klist:

::


Next, use the auth-client-config to configure the libpam-krb5 module to
request a ticket during login:

::

You will should now receive a ticket upon successful login
authentication.

Resources
---------

-  For more information on MIT's version of Kerberos, see the `MIT
   Kerberos <http://web.mit.edu/Kerberos/>`__ site.

-  The `Ubuntu Wiki
   Kerberos <https://help.ubuntu.com/community/Kerberos>`__ page has
   more details.

-  O'Reilly's `Kerberos: The Definitive
   Guide <http://oreilly.com/catalog/9780596004033/>`__ is a great
   reference when setting up Kerberos.

-  Also, feel free to stop by the *#ubuntu-server* and *#kerberos* IRC
   channels on `Freenode <http://freenode.net/>`__ if you have Kerberos
   questions.

Kerberos and LDAP
=================

Most people will not use Kerberos by itself; once an user is
authenticated (Kerberos), we need to figure out what this user can do
(authorization). And that would be the job of programs such as LDAP.

Replicating a Kerberos principal database between two servers can be
complicated, and adds an additional user database to your network.
Fortunately, MIT Kerberos can be configured to use an LDAP directory as
a principal database. This section covers configuring a primary and
secondary kerberos server to use OpenLDAP for the principal database.

    **Note**

    The examples presented here assume MIT Kerberos and OpenLDAP.

Configuring OpenLDAP
--------------------

First, the necessary *schema* needs to be loaded on an OpenLDAP server
that has network connectivity to the Primary and Secondary KDCs. The
rest of this section assumes that you also have LDAP replication
configured between at least two servers. For information on setting up
OpenLDAP see ?.

It is also required to configure OpenLDAP for TLS and SSL connections,
so that traffic between the KDC and LDAP server is encrypted. See ? for
details.

    **Note**

    ``cn=admin,cn=config`` is a user we created with rights to edit the
    ldap database. Many times it is the RootDN. Change its value to
    reflect your setup.

-  To load the schema into LDAP, on the LDAP server install the
   krb5-kdc-ldap package. From a terminal enter:

   ::

-  Next, extract the ``kerberos.schema.gz`` file:

   ::


-  The *kerberos* schema needs to be added to the *cn=config* tree. The
   procedure to add a new schema to slapd is also detailed in ?.

   First, create a configuration file named ``schema_convert.conf``, or
   a similar descriptive name, containing the following lines:

   ::

       include /etc/ldap/schema/core.schema
       include /etc/ldap/schema/collective.schema
       include /etc/ldap/schema/corba.schema
       include /etc/ldap/schema/cosine.schema
       include /etc/ldap/schema/duaconf.schema
       include /etc/ldap/schema/dyngroup.schema
       include /etc/ldap/schema/inetorgperson.schema
       include /etc/ldap/schema/java.schema
       include /etc/ldap/schema/misc.schema
       include /etc/ldap/schema/nis.schema
       include /etc/ldap/schema/openldap.schema
       include /etc/ldap/schema/ppolicy.schema
       include /etc/ldap/schema/kerberos.schema

   Create a temporary directory to hold the LDIF files:

   ::

   Now use slapcat to convert the schema files:

   ::

   Change the above file and path names to match your own if they are
   different.

   Edit the generated ``/tmp/cn\=kerberos.ldif`` file, changing the
   following attributes:

   ::

       dn: cn=kerberos,cn=schema,cn=config
       ...
       cn: kerberos

   And remove the following lines from the end of the file:

   ::

       structuralObjectClass: olcSchemaConfig
       entryUUID: 18ccd010-746b-102d-9fbe-3760cca765dc
       creatorsName: cn=config
       createTimestamp: 20090111203515Z
       entryCSN: 20090111203515.326445Z#000000#000#000000
       modifiersName: cn=config
       modifyTimestamp: 20090111203515Z

   The attribute values will vary, just be sure the attributes are
   removed.

   Load the new schema with ldapadd:

   ::

   Add an index for the *krb5principalname* attribute:

   ::


   Finally, update the Access Control Lists (ACL):

   ::


That's it, your LDAP directory is now ready to serve as a Kerberos
principal database.

Primary KDC Configuration
-------------------------

With OpenLDAP configured it is time to configure the KDC.

-  First, install the necessary packages, from a terminal enter:

   ::

-  Now edit ``/etc/krb5.conf`` adding the following options to under the
   appropriate sections:

   ::

       [libdefaults]
               default_realm = EXAMPLE.COM

       ...

       [realms]
               EXAMPLE.COM = {
                       kdc = kdc01.example.com
                       kdc = kdc02.example.com
                       admin_server = kdc01.example.com
                       admin_server = kdc02.example.com
                       default_domain = example.com
                       database_module = openldap_ldapconf
               }

       ...

       [domain_realm]
               .example.com = EXAMPLE.COM


       ...

       [dbdefaults]
               ldap_kerberos_container_dn = dc=example,dc=com

       [dbmodules]
               openldap_ldapconf = {
                       db_library = kldap
                       ldap_kdc_dn = "cn=admin,dc=example,dc=com"

                       # this object needs to have read rights on
                       # the realm container, principal container and realm sub-trees
                       ldap_kadmind_dn = "cn=admin,dc=example,dc=com"

                       # this object needs to have read and write rights on
                       # the realm container, principal container and realm sub-trees
                       ldap_service_password_file = /etc/krb5kdc/service.keyfile
                       ldap_servers = ldaps://ldap01.example.com ldaps://ldap02.example.com
                       ldap_conns_per_server = 5
               }

       **Note**

       Change *example.com*, *dc=example,dc=com*,
       *cn=admin,dc=example,dc=com*, and *ldap01.example.com* to the
       appropriate domain, LDAP object, and LDAP server for your
       network.

-  Next, use the kdb5\_ldap\_util utility to create the realm:

   ::

-  Create a stash of the password used to bind to the LDAP server. This
   password is used by the *ldap\_kdc\_dn* and *ldap\_kadmin\_dn*
   options in ``/etc/krb5.conf``:

   ::

-  Copy the CA certificate from the LDAP server:

   ::


   And edit ``/etc/ldap/ldap.conf`` to use the certificate:

   ::

       TLS_CACERT /etc/ssl/certs/cacert.pem

       **Note**

       The certificate will also need to be copied to the Secondary KDC,
       to allow the connection to the LDAP servers using LDAPS.

You can now add Kerberos principals to the LDAP database, and they will
be copied to any other LDAP servers configured for replication. To add a
principal using the kadmin.local utility enter:

::


There should now be krbPrincipalName, krbPrincipalKey, krbLastPwdChange,
and krbExtraData attributes added to the
*uid=steve,ou=people,dc=example,dc=com* user object. Use the kinit and
klist utilities to test that the user is indeed issued a ticket.

    **Note**

    If the user object is already created the *-x dn="..."* option is
    needed to add the Kerberos attributes. Otherwise a new *principal*
    object will be created in the realm subtree.

Secondary KDC Configuration
---------------------------

Configuring a Secondary KDC using the LDAP backend is similar to
configuring one using the normal Kerberos database.

First, install the necessary packages. In a terminal enter:

::

Next, edit ``/etc/krb5.conf`` to use the LDAP backend:

::

    [libdefaults]
            default_realm = EXAMPLE.COM

    ...

    [realms]
            EXAMPLE.COM = {
                    kdc = kdc01.example.com
                    kdc = kdc02.example.com
                    admin_server = kdc01.example.com
                    admin_server = kdc02.example.com
                    default_domain = example.com
                    database_module = openldap_ldapconf
            }

    ...

    [domain_realm]
            .example.com = EXAMPLE.COM

    ...

    [dbdefaults]
            ldap_kerberos_container_dn = dc=example,dc=com

    [dbmodules]
            openldap_ldapconf = {
                    db_library = kldap
                    ldap_kdc_dn = "cn=admin,dc=example,dc=com"

                    # this object needs to have read rights on
                    # the realm container, principal container and realm sub-trees
                    ldap_kadmind_dn = "cn=admin,dc=example,dc=com"

                    # this object needs to have read and write rights on
                    # the realm container, principal container and realm sub-trees
                    ldap_service_password_file = /etc/krb5kdc/service.keyfile
                    ldap_servers = ldaps://ldap01.example.com ldaps://ldap02.example.com
                    ldap_conns_per_server = 5
            }

Create the stash for the LDAP bind password:

::

Now, on the *Primary KDC* copy the ``/etc/krb5kdc/stash`` *Master Key*
stash to the Secondary KDC. Be sure to copy the file over an encrypted
connection such as scp, or on physical media.

::


    **Note**

    Again, replace *EXAMPLE.COM* with your actual realm.

Back on the *Secondary KDC*, (re)start the ldap server only,

::

Finally, start the krb5-kdc daemon:

::

Verify the two ldap servers (and kerberos by extension) are in sync.

You now have redundant KDCs on your network, and with redundant LDAP
servers you should be able to continue to authenticate users if one LDAP
server, one Kerberos server, or one LDAP and one Kerberos server become
unavailable.

Resources
---------

-  The `Kerberos Admin
   Guide <http://web.mit.edu/Kerberos/krb5-1.6/krb5-1.6.3/doc/krb5-admin.html#Configuring-Kerberos-with-OpenLDAP-back_002dend>`__
   has some additional details.

-  For more information on kdb5\_ldap\_util see `Section
   5.6 <http://web.mit.edu/Kerberos/krb5-1.6/krb5-1.6.3/doc/krb5-admin.html#Global-Operations-on-the-Kerberos-LDAP-Database>`__
   and the `kdb5\_ldap\_util man
   page <http://manpages.ubuntu.com/manpages/&distro-short-codename;/en/man8/kdb5_ldap_util.8.html>`__.

-  Another useful link is the `krb5.conf man
   page <http://manpages.ubuntu.com/manpages/&distro-short-codename;/en/man5/krb5.conf.5.html>`__.

-  Also, see the `Kerberos and
   LDAP <https://help.ubuntu.com/community/Kerberos#kerberos-ldap>`__
   Ubuntu wiki page.


