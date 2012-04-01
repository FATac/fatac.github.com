En aquest apartat s'indiquen totes les passes per tal de posar en marxa una instància de tota la pila de software i serveis necessaris per al funcionament d'una instància del projecte Arts Combinatòries.

Requisits
=========
Cal satisfer una sèrie de requisits previs que es detallen a continuació.

Sistema operatiu
-----------------
Actualment, degut a limitacions tecnològiques, el entorn del projecte només pot executar-se sobre màquines \*nix. En aquests documents suposarem una màquina basada en una distribució Ubuntu Linux 11.04 (Natty).

Software base i llibreries
--------------------------
 * GCC i eines de compilació
 * Python 2.6 (altres versions NO funcionen per Plone 4)
 * Python Distribute (a.k.a setuptools)
 * Python Imaging Library instal·lat per l'intèrpret de Python anterior (més informació en l'apartat corresponent)
 * Gawk
 * Git

Instal·lació de software base i llibreries
==========================================
Instal·lació de Python 2.6 i altres llibreries::
    
    sudo apt-get install python2.6 python-imaging wget build-essential python2.6-dev python-setuptools git-core

Instal·lació de Python Distribute::

    wget http://python-distribute.org/distribute_setup.py
    sudo python2.6 distribute_setup.py

Instal·lació de Gawk::

    sudo apt-get install gawk

Instal·lació d'Arts Combinatòries
=================================
Arts Combinatòries utilitza un sistema de montatge (descàrrega, compilació i configuració) automàtic anomenat `Buildout <http://www.buildout.org>`_.

Aquest sistema utilitza un fitxer (`buildout.cfg`) de configuració que contenen tota la informació necessària per a construir un entorn complet i configurat.

Per començar ens baixarem de Github el codi executant la següent ordre::

    git clone git://github.com/FATac/fatac.buildout.git

Fet això, ens ha d'haver creat un nou directori `fatac.buildout`. Accedim a ell::

    cd fatac.buildout

Executem el muntatge amb aquestes ordres::

    python2.6 bootstrap.py
    ./bin/buildout    

Un cop finalitzat, ens situem a la carpeta contenidora de la carpeta "fatac.buildout" i executem::

	useradd plone
	chown -R plone:plone fatac.buildout/

Instal·lació automàtica del lighttpd::

	apt-get install lighttpd

Cal afegir les aplicacions web del Tomcat (arxius .WAR a la carpeta "webapps"), que trobareu als llocs següents:

 - Serveis REST d'Arts Combinatòries: https://github.com/FATac/ArtsCombinatoriesRest/blob/develop/WebContent/WEB-INF/ArtsCombinatoriesRest.war
 - Motor de cerca Solr: http://lucene.apache.org/solr/ (a "Download")
 - Servidor OAI-PMH: http://code.google.com/p/oaicat/downloads/detail?name=oaicat.war

**Nota** Alguns serveis com Virtuoso i el servidor d'aplicacions Tomcat necessiten una configuració posterior que depén del vostre entorn i necessitats, tal i com es detalla en el apartat `Configuració` d'aquesta mateixa documentació.
    
Arrencada dels serveis relacionats amb Arts Combinatòries
------------------------------------------------------------

Els processos necessaris per al bon funcionament del servei són:

 - Plone (frontend)
 - Tomcat (API REST)
 - Virtuoso (BBDD)
 - Lighttpd (Servidor HTTP per l'streaming - opcional)

Plone::
    
    (directori plone)/bin/instance start
    
**Nota** La primera vegada cal crear des de http://urlservidor:8084/ el "Plone site" i escollir l'Add-on "fatac.theme".  

Tomcat::

    (directori tomcat)/bin/startup.sh

Virtuoso::

    cd (directori virtuoso)/var/lib/virtuoso/db/
    ../../../../bin/virtuoso-t -f &

Lighttpd::

    cd /etc/lighttpd/
    lighttpd -f lighttpd.conf

Serveis de vídeo
---------------------------------

La instal·lació dels serveis de vídeo consta de dues parts: una primera aplicació web Java i  després una llibreria nativa que ajuda en la codificació del vídeo.

1. Instal·lació del servei web

Bàsicament cal desplegar el fitxer TapiesWebServices.war dins del servidor d’aplicacions Tomcat. Aquest war ja disposa de una configuració de perfils predeterminada (fitxer profile.xml) de manera que no cal configurar res més. En tot cas, si calen afegir nous perfils per diferents formats d’àudio aquesta informació esta a la guia de configuració.

2. Instal·lació de la llibreria nativa Xuggler

Tot el software necessari esta preparat en un programa auto-instal·lador. Nomes cal executar-lo (normalment amb permisos d’administrador perquè pugui accedir a les carpetes necessàries.

::

	sudo sh ./xuggle-xuggler.4.0.1084-i686-pc-linux-gnu.sh


Cal confirmar la instal·lació escrivint ‘yes’ i podem deixar el directori d’instal.lació per defecte.

2.1 Comprovació de les llibreries natives.

A la carpeta /usr/local/xuggler/bin podem trobar els binaris del xuggler. Si executem per exemple el programa ffmpeg hauria de surtir la informació de versió:

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


En cas de que el sistema li falti alguna llibreria nativa, caldrà instal.lar-la depenent de la distribució de Linux que es faci servir.

En el cas de la distribució Ubuntu 11.10 per equips de 32 bits no cal afegir cap llibreria addicional. 

En el cas de la Ubuntu 11.10 per equips de 64 bits, cal afegir les llibreries de compatibilitat.

::
	
	sudo apt-get install ia32-libs


2.2  Afegir el Xuggler al Tomcat

Copiem de /usr/local/xuggler/share/java/jars totes les dependències a la carpeta %TOMCAT_HOME%/lib . Més concretament són els fitxers:

- logback-classic.jar
- logback-core.jar
- commons-cli.jar
- slf4j-api.jar
- xuggle-xuggler.jar

Per últim cal configurar el tomcat per trobi les llibreries Xuggler. afegim les rutes d’accés al script d’arranc del tomcat %TOMCAT_HOME%/bin/startup.sh:

::

	export XUGGLE_HOME=/usr/local/xuggler
	export LD_LIBRARY_PATH=$XUGGLE_HOME/lib:$LD_LIBRARY_PATH
	export PATH=$XUGGLE_HOME/bin:$PATH
