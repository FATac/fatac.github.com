This section describes the steps required to build and run an instance of the software and services stack of Arts Combinatòries.

Requirements
============
You should meet all the requirements described below.

Operating System
-----------------
Due to technological limitations, the project is able to run on \*nix based machines. From now on all the examples shown will be based under a assumption of having a Ubuntu Linux 11.04 (Natty) distribution.

Basic Software
--------------
 * GCC compiler suite to build native Python extensions (Zope contains C code for optimized parts)
 * Python 2.6 (other versions are *not* ok for Plone 4)
 * Python Imaging Library installed for your Python interpreter (more below)
 * Python `Distribute <http://pypi.python.org/pypi/distribute>`_ installation tool, provided by your operating system
  or installed by hand
 * Gawk
 * Git

Install basic software
======================
GCC, Python 2.6 and additional software and libraries::
    
    sudo apt-get install python2.6 python-imaging wget build-essential python2.6-dev python-setuptools git-core

Python Distribute::

    wget http://python-distribute.org/distribute_setup.py
    sudo python2.6 distribute_setup.py

Gawk::

    sudo apt-get install gawk

Arts Combinatòries install
===========================
`Buildout <http://www.buildout.org>`_ is a tool which automatically downloads, installs and configures Python and other software. Arts Combinatòries uses buildout.

This build system uses a file (`buildout.cfg`) which provides all the configuration information to build the complete stack of required software.

We should download the buildout code from Github::

    git clone git://github.com/FATac/fatac.buildout.git

Access to the recently created folder `fatac.buildout`::

    cd fatac.buildout

Execute the buildout process::

    python2.6 bootstrap.py
    ./bin/buildout

..note::

	Some services such as Virtuoso and Tomcat need additional configuration which depends on your needs and environment, this is explained further in 'Configuration' section.

Run the processes of the project software stack
------------------------------------------------

These are the processes needed by Arts Combinatòries:

 - Plone (frontend)
 - Tomcat (API REST)
 - Virtuoso (BBDD)
 - Lighttpd (HTTP server - optional)

Plone::
    
    (plone path)/bin/instance start

Tomcat::

    (tomcat path)/bin/startup.sh

Virtuoso::

    cd (virtuoso path)/var/lib/virtuoso/db/
    ../../../../bin/virtuoso-t -f &
    
Lighttpd::

	cd /etc/lighttpd/
	lighttpd -D -f lighttpd.conf