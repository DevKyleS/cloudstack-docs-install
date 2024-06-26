.. Licensed to the Apache Software Foundation (ASF) under one
   or more contributor license agreements.  See the NOTICE file
   distributed with this work for additional information#
   regarding copyright ownership.  The ASF licenses this file
   to you under the Apache License, Version 2.0 (the
   "License"); you may not use this file except in compliance
   with the License.  You may obtain a copy of the License at
   http://www.apache.org/licenses/LICENSE-2.0
   Unless required by applicable law or agreed to in writing,
   software distributed under the License is distributed on an
   "AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
   KIND, either express or implied.  See the License for the
   specific language governing permissions and limitations
   under the License.


Building from Source
====================

The official CloudStack release is always in source code form. You will
likely be able to find "convenience binaries," the source is the
canonical release. In this section, we'll cover acquiring the source
release and building that so that you can deploy it using Maven or
create Debian packages or RPMs.

Note that building and deploying directly from source is typically not
the most efficient way to deploy an IaaS. However, we will cover that
method as well as building RPMs or Debian packages for deploying
CloudStack.

The instructions here are likely version-specific. That is, the method
for building from source for the 4.7.x series is different from the
4.2.x series.

If you are working with a unreleased version of CloudStack, see the
INSTALL.md file in the top-level directory of the release.


Getting the release
-------------------

You can download the latest CloudStack release from the `Apache
CloudStack project download page 
<http://cloudstack.apache.org/downloads.html>`_.

Prior releases are available via archive.apache.org as well. See the
downloads page for more information on archived releases.

You'll notice several links under the 'Latest release' section. A link
to a file ending in ``tar.bz2``, as well as a PGP/GPG signature, MD5,
and SHA512 file.

-  The ``tar.bz2`` file contains the Bzip2-compressed tarball with the
   source code.

-  The ``.asc`` file is a detached cryptographic signature that can be
   used to help verify the authenticity of the release.

-  The ``.md5`` file is an MD5 hash of the release to aid in verify the
   validity of the release download.

-  The ``.sha`` file is a SHA512 hash of the release to aid in verify
   the validity of the release download.


Verifying the downloaded release
--------------------------------

There are a number of mechanisms to check the authenticity and validity
of a downloaded release.


Getting the KEYS
~~~~~~~~~~~~~~~~

To enable you to verify the GPG signature, you will need to download the
`KEYS <http://www.apache.org/dist/cloudstack/KEYS>`_ file.

You next need to import those keys, which you can do by running:

.. sourcecode:: bash

   $ wget http://www.apache.org/dist/cloudstack/KEYS
   $ gpg --import KEYS


GPG
~~~

The CloudStack project provides a detached GPG signature of the release.
To check the signature, run the following command:

.. sourcecode:: bash

   $ gpg --verify apache-cloudstack-4.11.0.0-src.tar.bz2.asc

If the signature is valid you will see a line of output that contains
'Good signature'.


MD5
~~~

In addition to the cryptographic signature, CloudStack has an MD5
checksum that you can use to verify the download matches the release.
You can verify this hash by executing the following command:

.. sourcecode:: bash

   $ gpg --print-md MD5 apache-cloudstack-4.11.0.0-src.tar.bz2 | diff - apache-cloudstack-4.11.0.0-src.tar.bz2.md5

If this successfully completes you should see no output. If there is any
output from them, then there is a difference between the hash you
generated locally and the hash that has been pulled from the server.


SHA512
~~~~~~

In addition to the MD5 hash, the CloudStack project provides a SHA512
cryptographic hash to aid in assurance of the validity of the downloaded
release. You can verify this hash by executing the following command:

.. sourcecode:: bash

   $ gpg --print-md SHA512 apache-cloudstack-4.11.0.0-src.tar.bz2 | diff - apache-cloudstack-4.11.0.0-src.tar.bz2.sha

If this command successfully completes you should see no output. If
there is any output from them, then there is a difference between the
hash you generated locally and the hash that has been pulled from the
server.


Prerequisites for building Apache CloudStack
--------------------------------------------

There are a number of prerequisites needed to build CloudStack. This
document assumes compilation on a Linux system that uses RPMs or DEBs
for package management.

You will need, at a minimum, the following to compile CloudStack:

#. Maven (version 3)

#. Java (Java 8/OpenJDK 1.8)

#. Apache Web Services Common Utilities (ws-commons-util)

#. MySQL

#. MySQLdb (provides Python database API)

#. genisoimage

#. rpmbuild or dpkg-dev


Extracting source
-----------------

Extracting the CloudStack release is relatively simple and can be done
with a single command as follows:

.. sourcecode:: bash

   $ tar -jxvf apache-cloudstack-4.11.0.0-src.tar.bz2

You can now move into the directory:

.. sourcecode:: bash

   $ cd ./apache-cloudstack-4.11.0.0-src

Install new MySQL connector
---------------------------

Install Python MySQL connector using the official MySQL packages repository.


MySQL connector APT repository
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Install the following package provided by MySQL to enable official repositories:

.. sourcecode:: bash

   wget http://dev.mysql.com/get/mysql-apt-config_0.7.3-1_all.deb
   sudo dpkg -i mysql-apt-config_0.7.3-1_all.deb

Make sure to activate the repository for MySQL connectors.

.. sourcecode:: bash

   sudo apt-get update
   sudo apt-get install mysql-connector-python   


MySQL connector RPM repository
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Add a new yum repo ``/etc/yum.repos.d/mysql.repo``:

.. sourcecode:: bash

   [mysql-community]
   name=MySQL Community connectors
   baseurl=http://repo.mysql.com/yum/mysql-connectors-community/el/$releasever/$basearch/
   enabled=1
   gpgcheck=1

Import GPG public key from MySQL:

.. sourcecode:: bash

   rpm --import http://repo.mysql.com/RPM-GPG-KEY-mysql

Install mysql-connector

.. sourcecode:: bash

   yum install mysql-connector-python


Building DEB packages
---------------------

In addition to the bootstrap dependencies, you'll also need to install
several other dependencies. Note that we recommend using Maven 3.

.. sourcecode:: bash

   $ sudo apt-get update
   $ sudo apt-get install python-software-properties
   $ sudo apt-get update
   $ sudo apt-get install debhelper openjdk-8-jdk libws-commons-util-java genisoimage libcommons-codec-java libcommons-httpclient-java liblog4j1.2-java maven

While we have defined, and you have presumably already installed the
bootstrap prerequisites, there are a number of build time prerequisites
that need to be resolved. CloudStack uses maven for dependency
resolution. You can resolve the buildtime depdencies for CloudStack by
running:

.. sourcecode:: bash

   $ mvn -P deps

Now that we have resolved the dependencies we can move on to building
CloudStack and packaging them into DEBs by issuing the following
command.

.. sourcecode:: bash

   $ dpkg-buildpackage -uc -us

This command will build the following debian packages. You should have
all of the following:

.. sourcecode:: bash

   cloudstack-common-4.11.0.0.amd64.deb
   cloudstack-management-4.11.0.0.amd64.deb
   cloudstack-agent-4.11.0.0.amd64.deb
   cloudstack-usage-4.11.0.0.amd64.deb
   cloudstack-cli-4.11.0.0.amd64.deb


Setting up an APT repo
~~~~~~~~~~~~~~~~~~~~~~

After you've created the packages, you'll want to copy them to a system
where you can serve the packages over HTTP. You'll create a directory
for the packages and then use ``dpkg-scanpackages`` to create
``Packages.gz``, which holds information about the archive structure.
Finally, you'll add the repository to your system(s) so you can install
the packages using APT.

The first step is to make sure that you have the **dpkg-dev** package
installed. This should have been installed when you pulled in the
**debhelper** application previously, but if you're generating
``Packages.gz`` on a different system, be sure that it's installed there
as well.

.. sourcecode:: bash

   $ sudo apt-get install dpkg-dev

The next step is to copy the DEBs to the directory where they can be
served over HTTP. We'll use ``/var/www/cloudstack/repo`` in the
examples, but change the directory to whatever works for you.

.. sourcecode:: bash

   $ sudo mkdir -p /var/www/cloudstack/repo/binary
   $ sudo cp *.deb /var/www/cloudstack/repo/binary
   $ cd /var/www/cloudstack/repo/binary
   $ sudo sh -c 'dpkg-scanpackages . /dev/null | tee Packages | gzip -9 > Packages.gz'

.. note:: 
   You can safely ignore the warning about a missing override file.

Now you should have all of the DEB packages and ``Packages.gz`` in the
``binary`` directory and available over HTTP. (You may want to use
``wget`` or ``curl`` to test this before moving on to the next step.)


Configuring your machines to use the APT repository
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Now that we have created the repository, you need to configure your
machine to make use of the APT repository. You can do this by adding a
repository file under ``/etc/apt/sources.list.d``. Use your preferred
editor to create ``/etc/apt/sources.list.d/cloudstack.list`` with this
line:

.. sourcecode:: bash

   deb http://server.url/cloudstack/repo/binary ./

Now that you have the repository info in place, you'll want to run
another update so that APT knows where to find the CloudStack packages.

.. sourcecode:: bash

   $ sudo apt-get update

You can now move on to the instructions under Install on Ubuntu.


Building RPMs from Source
-------------------------

As mentioned previously in `“Prerequisites for building Apache CloudStack” 
<#prerequisites-for-building-apache-cloudstack>`_, you will need to install 
several prerequisites before you can build packages for CloudStack. Here we'll
assume you're working with a 64-bit build of CentOS or Red Hat Enterprise 
Linux.

.. sourcecode:: bash

   # yum groupinstall "Development Tools"

.. sourcecode:: bash

   # yum install java-1.8.0-openjdk-devel.x86_64 genisoimage mysql mysql-server ws-commons-util MySQL-python createrepo

Next, you'll need to install build-time dependencies for CloudStack with
Maven. We're using Maven 3, so you'll want to grab `Maven 3.0.5 (Binary tar.gz)
<http://maven.apache.org/download.cgi>`_ and uncompress it in
your home directory (or whatever location you prefer):

.. sourcecode:: bash

   $ cd ~
   $ tar zxvf apache-maven-3.0.5-bin.tar.gz

.. sourcecode:: bash

   $ export PATH=~/apache-maven-3.0.5/bin:$PATH

Maven also needs to know where Java is, and expects the JAVA\_HOME
environment variable to be set:

.. sourcecode:: bash

   $ export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk.x86_64

Verify that Maven is installed correctly:

.. sourcecode:: bash

   $ mvn --version

You probably want to ensure that your environment variables will survive
a logout/reboot. Be sure to update ``~/.bashrc`` with the PATH and
JAVA\_HOME variables.

Building RPMs for CloudStack is fairly simple. Assuming you already have
the source downloaded and have uncompressed the tarball into a local
directory, you're going to be able to generate packages in just a few
minutes.

.. note::
   Packaging has Changed. If you've created packages for CloudStack 
   previously, you should be aware that the process has changed considerably 
   since the project has moved to using Apache Maven. Please be sure to follow 
   the steps in this section closely.


Generating RPMS
~~~~~~~~~~~~~~~

Now that we have the prerequisites and source, you will cd to the 
`packaging/` directory.

.. sourcecode:: bash

   $ cd packaging/

Generating RPMs is done using the ``package.sh`` script:

.. sourcecode:: bash

   $ ./package.sh -d centos63
   
For other supported options(like centos7), run ``./package.sh --help``

That will run for a bit and then place the finished packages in
``dist/rpmbuild/RPMS/x86_64/``.

You should see the following RPMs in that directory:

.. sourcecode:: bash

   cloudstack-agent-4.11.0.0.el6.x86_64.rpm
   cloudstack-cli-4.11.0.0.el6.x86_64.rpm
   cloudstack-common-4.11.0.0.el6.x86_64.rpm
   cloudstack-management-4.11.0.0.el6.x86_64.rpm
   cloudstack-usage-4.11.0.0.el6.x86_64.rpm


Creating a yum repo
^^^^^^^^^^^^^^^^^^^

While RPMs is a useful packaging format - it's most easily consumed from
Yum repositories over a network. The next step is to create a Yum Repo
with the finished packages:

.. sourcecode:: bash

   $ mkdir -p ~/tmp/repo

.. sourcecode:: bash

   $ cd ../..
   $ cp dist/rpmbuild/RPMS/x86_64/*rpm ~/tmp/repo/

.. sourcecode:: bash

   $ createrepo ~/tmp/repo

The files and directories within ``~/tmp/repo`` can now be uploaded to a
web server and serve as a yum repository.


Configuring your systems to use your new yum repository
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Now that your yum repository is populated with RPMs and metadata we need
to configure the machines that need to install CloudStack. Create a file
named ``/etc/yum.repos.d/cloudstack.repo`` with this information:

.. sourcecode:: bash

   [apache-cloudstack]
   name=Apache CloudStack
   baseurl=http://webserver.tld/path/to/repo
   enabled=1
   gpgcheck=0

Completing this step will allow you to easily install CloudStack on a
number of machines across the network.


Building Non-OSS
----------------

If you need support for the VMware, NetApp, F5, NetScaler, SRX, or any
other non-Open Source Software (nonoss) plugins, you'll need to download
a few components on your own and follow a slightly different procedure
to build from source.

.. warning::
   Some of the plugins supported by CloudStack cannot be distributed with 
   CloudStack for licensing reasons. In some cases, some of the required 
   libraries/JARs are under a proprietary license. In other cases, the 
   required libraries may be under a license that's not compatible with 
   `Apache's licensing guidelines for third-party products 
   <http://www.apache.org/legal/resolved.html#category-x>`_.

#. To build the Non-OSS plugins, you'll need to have the requisite JARs
   installed under the ``deps`` directory.

   Because these modules require dependencies that can't be distributed
   with CloudStack you'll need to download them yourself. Links to the
   most recent dependencies are listed on the `*How to build CloudStack* 
   <https://cwiki.apache.org/confluence/display/CLOUDSTACK/How+to+build+CloudStack>`_
   page on the wiki.

#. You may also need to download
   `vhd-util <http://download.cloudstack.org/tools/vhd-util>`_,
   which was removed due to licensing issues. You'll copy vhd-util to
   the ``scripts/vm/hypervisor/xenserver/`` directory.

#. Once you have all the dependencies copied over, you'll be able to
   build CloudStack with the ``noredist`` option:

.. sourcecode:: bash

   $ mvn clean
   $ mvn install -Dnoredist

#. Once you've built CloudStack with the ``noredist`` profile, you can
   package it using the `“Building RPMs from Source” <#building-rpms-from-source>`_ 
   or `“Building DEB packages” <#building-deb-packages>`_ instructions.

