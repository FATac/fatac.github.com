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

Executem el montatge amb aquestes ordres::

    python2.6 bootstrap.py
    ./bin/buildout

Esperem fins a que finalitzi. Un cop finalitzar, tindrem tots els serveis llestos per ser arrencats.

..note::

    Alguns serveis com Virtuoso i el servidor d'aplicacions Tomcat necessiten una configuració posterior que depén del vostre entorn i necessitats, tal i com es detalla en el apartat `Configuració` d'aquesta mateixa documentació.

Arrencada dels serveis relacionats amb Arts Combinatòries
----------------------------------------------------------

Els processos necessaris per al bon funcionament del servei són:

 - Plone (frontend)
 - Tomcat (API REST)
 - Virtuoso (BBDD)
 - Lighttpd (Servidor HTTP per l'streaming - opcional)

Plone::
    
    (directori plone)/bin/instance start

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

**(i2CAT) TODO: Documentar la instal·lació dels serveis de transcodificació de vídeo**
