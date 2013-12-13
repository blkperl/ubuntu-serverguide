Version Control System
======================

Version control is the art of managing changes to information. It has
long been a critical tool for programmers, who typically spend their
time making small changes to software and then undoing those changes the
next day. But the usefulness of version control software extends far
beyond the bounds of the software development world. Anywhere you can
find people using computers to manage information that changes often,
there is room for version control.

Bazaar
======

Bazaar is a new version control system sponsored by Canonical, the
commercial company behind Ubuntu. Unlike Subversion and CVS that only
support a central repository model, Bazaar also supports *distributed
version control*, giving people the ability to collaborate more
efficiently. In particular, Bazaar is designed to maximize the level of
community participation in open source projects.

Installation
------------

At a terminal prompt, enter the following command to install bzr:

::

Configuration
-------------

To introduce yourself to bzr, use the *whoami* command like this:

::

Learning Bazaar
---------------

Bazaar comes with bundled documentation installed into
/usr/share/doc/bzr/html by default. The tutorial is a good place to
start. The bzr command also comes with built-in help:

::

To learn more about the *foo* command:

::

Launchpad Integration
---------------------

While highly useful as a stand-alone system, Bazaar has good, optional
integration with `Launchpad <https://launchpad.net/>`__, the
collaborative development system used by Canonical and the broader open
source community to manage and extend Ubuntu itself. For information on
how Bazaar can be used with Launchpad to collaborate on open source
projects, see
`http://bazaar-vcs.org/LaunchpadIntegration <http://bazaar-vcs.org/LaunchpadIntegration/>`__.

Git
===

Git is an open source distributed version control system originally
developped by Linus Torvalds to support the development of the linux
kernel. Every Git working directory is a full-fledged repository with
complete history and full version tracking capabilities, not dependent
on network access or a central server.

Installation
------------

The git version control system is installed with the following command

::

Configuration
-------------

Every git user should first introduce herself to git, by running these
two commands:

::


Basic usage
-----------

The above is already sufficient to use git in a distributed and secure
way, provided users have access to the machine assuming the server role
via SSH. On the server machine, creating a new repository can be done
with

::

    **Note**

    This creates a bare repository, that cannot be used to edit files
    directly. If you would rather have a working copy of the contents of
    the repository on the server, ommit the *--bare* option.

Any client with ssh access to the machine can from then on clone the
repository with

::

Once cloned to the client's machine, the client can edit files, then
commit and share them with:

::




Installing a gitolite server
----------------------------

While the above is sufficient to create, clone and edit repositories,
users wanting to install git on a server will most likely want to have
git work like a more traditional source control management server, with
multiple users and access rights management. The suggested solution is
to install gitolite with the following command:

::

Gitolite configuration
----------------------

Configuration of the gitolite server is a little different that most
other servers on Unix-like systems. Instead of the traditional
configuration files in /etc/, gitolite stores its configuration in a git
repository. The first step to configuring a new installation is
therefore to allow access to the configuration repository.

First of all, let's create a user for gitolite to be accessed as.

::

Now we want to let gitolite know about the repository administrator's
public SSH key. This assumes that the current user is the repository
administrator.

::

Let's switch to the git user and import the administrator's key into
gitolite.

::


Gitolite will allow you to make initial changes to its configuration
file during the setup process. You can now clone and modify the gitolite
configuration repository from your administrator user (the user whose
public SSH key you imported). Switch back to that user, then clone the
configuration repository:

::



The gitolite-admin contains to subdirectories, "conf" and "keydir". The
configuration files are in the conf dir, and the keydir directory
contains the list of user's public SSH keys.

Managing gitolite users and repositories
----------------------------------------

Adding new users to gitolite is simple: just obtain their public SSH key
and add it to the keydir directory as $DESIRED\_USER\_NAME.pub. Note
that the the gitolite usernames don't have to match the system usernames
- they are only used in the gitolite configuration file to manage access
control. Similarly, users are deleted by deleting their public key file.
After each change, do not forget to commit the changes to git, and push
the changes back to the server with

::


Reporitories are managed by editing the conf/gitolite.conf file. The
syntax is space separated, and simply specifies the list of repositories
followed by some access rules. The following is a default example

::

    repo    gitolite-admin
            RW+     =   admin
            R       =   alice

    repo    project1
            RW+     =   alice
            RW      =   bob
            R       =   denise

Using your server
-----------------

To use the newly created server, users have to have the gitolite admin
import their public key into the gitolite configuration repository, they
can then access any project they have access to with the following
command:

::

Or add the server's project as a remote for an existing git repostory:

::

Subversion
==========

Subversion is an open source version control system. Using Subversion,
you can record the history of source files and documents. It manages
files and directories over time. A tree of files is placed into a
central repository. The repository is much like an ordinary file server,
except that it remembers every change ever made to files and
directories.

Installation
------------

To access Subversion repository using the HTTP protocol, you must
install and configure a web server. Apache2 is proven to work with
Subversion. Please refer to the HTTP subsection in the Apache2 section
to install and configure Apache2. To access the Subversion repository
using the HTTPS protocol, you must install and configure a digital
certificate in your Apache 2 web server. Please refer to the HTTPS
subsection in the Apache2 section to install and configure the digital
certificate.

To install Subversion, run the following command from a terminal prompt:

::

Server Configuration
--------------------

This step assumes you have installed above mentioned packages on your
system. This section explains how to create a Subversion repository and
access the project.

Create Subversion Repository
~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The Subversion repository can be created using the following command
from a terminal prompt:

::

Importing Files
~~~~~~~~~~~~~~~

Once you create the repository you can *import* files into the
repository. To import a directory, enter the following from a terminal
prompt:

::

Access Methods
--------------

Subversion repositories can be accessed (checked out) through many
different methods --on local disk, or through various network protocols.
A repository location, however, is always a URL. The table describes how
different URL schemes map to the available access methods.

+------------+---------------------------------------------------------------+
| Schema     | Access Method                                                 |
+============+===============================================================+
| file://    | direct repository access (on local disk)                      |
+------------+---------------------------------------------------------------+
| http://    | Access via WebDAV protocol to Subversion-aware Apache2 web    |
|            | server                                                        |
+------------+---------------------------------------------------------------+
| https://   | Same as http://, but with SSL encryption                      |
+------------+---------------------------------------------------------------+
| svn://     | Access via custom protocol to an svnserve server              |
+------------+---------------------------------------------------------------+
| svn+ssh:// | Same as svn://, but through an SSH tunnel                     |
+------------+---------------------------------------------------------------+

Table: Access Methods

In this section, we will see how to configure Subversion for all these
access methods. Here, we cover the basics. For more advanced usage
details, refer to the `svn book <http://svnbook.red-bean.com/>`__.

Direct repository access (file://)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

This is the simplest of all access methods. It does not require any
Subversion server process to be running. This access method is used to
access Subversion from the same machine. The syntax of the command,
entered at a terminal prompt, is as follows:

::

or

::

    **Note**

    If you do not specify the hostname, there are three forward slashes
    (///) -- two for the protocol (file, in this case) plus the leading
    slash in the path. If you specify the hostname, you must use two
    forward slashes (//).

The repository permissions depend on filesystem permissions. If the user
has read/write permission, he can checkout from and commit to the
repository.

Access via WebDAV protocol (http://)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

To access the Subversion repository via WebDAV protocol, you must
configure your Apache 2 web server. Add the following snippet between
the *<VirtualHost>* and *</VirtualHost>* elements in
``/etc/apache2/sites-available/default``, or another VirtualHost file:

::

     <Location /svn>
      DAV svn
      SVNParentPath /path/to/repos
      AuthType Basic
      AuthName "Your repository name"
      AuthUserFile /etc/subversion/passwd
      Require valid-user
     </Location> 

    **Note**

    The above configuration snippet assumes that Subversion repositories
    are created under ``/path/to/repos`` directory using ``svnadmin``
    command and that the HTTP user has sufficent access rights to the
    files (see below). They can be accessible using
    ``http://hostname/svn/repos_name`` url.

Changing the apache configuration like the above requires to reload the
service with the following command

::

        

To import or commit files to your Subversion repository over HTTP, the
repository should be owned by the HTTP user. In Ubuntu systems, the HTTP
user is ``www-data``. To change the ownership of the repository files
enter the following command from terminal prompt:

::

        

    **Note**

    By changing the ownership of repository as ``www-data`` you will not
    be able to import or commit files into the repository by running
    ``svn import
            file:///`` command as any user other than ``www-data``.

Next, you must create the ``/etc/subversion/passwd`` file that will
contain user authentication details. To create a file issue the
following command at a command prompt (which will create the file and
add the first user):

::

To add additional users omit the *"-c"* option as this option replaces
the old file. Instead use this form:

::

This command will prompt you to enter the password. Once you enter the
password, the user is added. Now, to access the repository you can run
the following command:

::

    **Warning**

    The password is transmitted as plain text. If you are worried about
    password snooping, you are advised to use SSL encryption. For
    details, please refer next section.

Access via WebDAV protocol with SSL encryption (https://)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Accessing Subversion repository via WebDAV protocol with SSL encryption
(https://) is similar to http:// except that you must install and
configure the digital certificate in your Apache2 web server. To use SSL
with Subversion add the above Apache2 configuration to
``/etc/apache2/sites-available/default-ssl``. For more information on
setting up Apache2 with SSL see ?.

You can install a digital certificate issued by a signing authority.
Alternatively, you can install your own self-signed certificate.

This step assumes you have installed and configured a digital
certificate in your Apache 2 web server. Now, to access the Subversion
repository, please refer to the above section! The access methods are
exactly the same, except the protocol. You must use https:// to access
the Subversion repository.

Access via custom protocol (svn://)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Once the Subversion repository is created, you can configure the access
control. You can edit the ``
                    /path/to/repos/project/conf/svnserve.conf`` file to
configure the access control. For example, to set up authentication, you
can uncomment the following lines in the configuration file:

::

    # [general]
    # password-db = passwd

After uncommenting the above lines, you can maintain the user list in
the passwd file. So, edit the file ``passwd
                    `` in the same directory and add the new user. The
syntax is as follows:

::

    username = password

For more details, please refer to the file.

Now, to access Subversion via the svn:// custom protocol, either from
the same machine or a different machine, you can run svnserver using
svnserve command. The syntax is as follows:

::

    $ svnserve -d --foreground -r /path/to/repos
    # -d -- daemon mode
    # --foreground -- run in foreground (useful for debugging)
    # -r -- root of directory to serve

    For more usage details, please refer to:
    $ svnserve --help

Once you run this command, Subversion starts listening on default port
(3690). To access the project repository, you must run the following
command from a terminal prompt:

::

Based on server configuration, it prompts for password. Once you are
authenticated, it checks out the code from Subversion repository. To
synchronize the project repository with the local copy, you can run the
``update`` sub-command. The syntax of the command, entered at a terminal
prompt, is as follows:

::

For more details about using each Subversion sub-command, you can refer
to the manual. For example, to learn more about the co (checkout)
command, please run the following command from a terminal prompt:

::

Access via custom protocol with SSH encryption (svn+ssh://)
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

The configuration and server process is same as in the svn:// method.
For details, please refer to the above section. This step assumes you
have followed the above step and started the Subversion server using
svnserve command.

It is also assumed that the ssh server is running on that machine and
that it is allowing incoming connections. To confirm, please try to
login to that machine using ssh. If you can login, everything is
perfect. If you cannot login, please address it before continuing
further.

The svn+ssh:// protocol is used to access the Subversion repository
using SSL encryption. The data transfer is encrypted using this method.
To access the project repository (for example with a checkout), you must
use the following command syntax:

::

        

    **Note**

    You must use the full path (/path/to/repos/project) to access the
    Subversion repository using this access method.

Based on server configuration, it prompts for password. You must enter
the password you use to login via ssh. Once you are authenticated, it
checks out the code from the Subversion repository.

References
==========

-  `Bazaar Home Page <http://bazaar.canonical.com/en/>`__

-  `Launchpad <https://launchpad.net/>`__

-  `Git homepage <http://git-scm.com>`__

-  `Gitolite <https://github.com/sitaramc/gitolite>`__

-  `Subversion Home Page <http://subversion.tigris.org/>`__

-  `Subversion Book <http://svnbook.red-bean.com/>`__

-  `Easy Bazaar Ubuntu Wiki
   page <https://help.ubuntu.com/community/EasyBazaar>`__

-  `Ubuntu Wiki Subversion
   page <https://help.ubuntu.com/community/Subversion>`__


