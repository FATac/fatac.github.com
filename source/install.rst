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
    
This will process take some time.
    
Automatic installation of lighttpd::

	apt-get install lighttpd
    
Following Tomcat web applications (.war files) must be added in "webapps" directory:

 - Arts Combinatòries REST services: https://github.com/FATac/ArtsCombinatoriesRest/blob/develop/WebContent/WEB-INF/ArtsCombinatoriesRest.war
 - Solr search engine: http://lucene.apache.org/solr/ (in "Download")
 - OAI-PMH Server: http://code.google.com/p/oaicat/downloads/detail?name=oaicat.war

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
	lighttpd -f lighttpd.conf


Video services
-----------------------------

Video services install is made of two parts: first, a java web application and second, a native library that helps on video coding.

1. Web service installation 

Basically, you have to deploy file TapiesWebServices.war on Tomcat application server. This war already have an initial profile configuration (profile.xml file), so no additioanl configuration has to be done. In any case, if new profiles have to be added for supporting other video formats, this information is provided in configuration guide.

2. Xuggler native library installation 

All necessary software is ready in an auto-install program. You only have to execute it (usually with administrator rights to be able to access to the required directories).

::

	sudo sh ./xuggle-xuggler.4.0.1084-i686-pc-linux-gnu.sh

You have to confirm installation by writing 'yes' and you can leave installation directory as default.

2.1 Native libraries checking 

In directory /usr/local/xuggler/bin you can find Xuggler binaries. When executing the ffmpeg binary you will get its version information: 

::

	ffmpeg version 0.8.git-xuggle-4.0.revision.sh, Copyright (c) 2000-2011 the FFmpeg developers
	  built on Nov 21 2011 09:31:08 with gcc 4.6.1
	  configuration: --prefix=/usr/local/xuggler --extra-version=xuggle-4.0.revision.sh --extra-cflags=-I/home/javi/Desktop/xuggle-xuggler-modified/build/native/i686-pc-linux-gnu/captive/usr/local/xuggler/include --extra-ldflags=-L/home/javi/Desktop/xuggle-xuggler-modified/build/native/i686-pc-linux-gnu/captive/usr/local/xuggler/lib --enable-shared --enable-gpl --enable-nonfree --enable-version3 --enable-libx264 --enable-libmp3lame --enable-libvorbis --enable-libtheora --enable-libspeex --enable-libvpx --enable-libopencore-amrnb --enable-libopencore-amrwb --enable-pthreads
	  libavutil    51. 11. 0 / 51. 11. 0
	  libavcodec   53.  8. 0 / 53.  8. 0
	  libavformat  53.  6. 0 / 53.  6. 0
	  libavdevice  53.  2. 0 / 53.  2. 0
	  libavfilter   2. 27. 0 /  2. 27. 0
	  libswscale    2.  0. 0 /  2.  0. 0
	  libpostproc  51.  2. 0 / 51.  2. 0
	Use -h to get full help or, even better, run 'man ffmpeg'

In case a native library is missing, you should install it. This will depend on the linux distribution being used.

For Ubuntu 11.10 32 bits, no additional libraries has to be installed. 

For Ubuntu 11.10 64 bits, compatibility libraries package has to be installed. 

::
	
	sudo apt-get install ia32-libs


2.2  Install Xuggler support to Tomcat 

Copy all dependences from directory /usr/local/xuggler/share/java/jars to directory %TOMCAT_HOME%/lib . More specifically the files:

- logback-classic.jar
- logback-core.jar
- commons-cli.jar
- slf4j-api.jar
- xuggle-xuggler.jar

Finally, you have to configure environment variables to allow Tomcat to find Xuggler libraries. Add the following lines to Tomcat startup file (%TOMCAT_HOME%/bin/startup.sh):

::

	export XUGGLE_HOME=/usr/local/xuggler
	export LD_LIBRARY_PATH=$XUGGLE_HOME/lib:$LD_LIBRARY_PATH
	export PATH=$XUGGLE_HOME/bin:$PATH
