.. FAT Arts Combinatòries documentation master file, created by
   sphinx-quickstart on Tue May 31 12:39:26 2011.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

Introducció
======================================================================================

Arts Combinatòries és una Web Semàntica de propòsit general per a continguts culturals (és a dir audiovisuals). Com a tal, proporciona patrons completament configurables per permetre a qualsevol usuari o institució allotjar les seves dades semàntiques (OWL/RDF) i relacionar-les amb els seus arxius. Aquesta documentació aprofundeix en la configuració de patrons i altres paràmetres útils per a la personalització. 

Característiques
-----------------------

Les funcions clau són:

- Conjunt de serveis REST que proporcionen una completa interacció amb les dades del sistema.
- Allotjament dels media (audio, vídeo, imatge i text) i funcions per la conversió multi-format.
- Protecció legal completament configurable per adaptar-se a qualsevol llicència d'objectes culturals.
- Suport per a usuaris i grups, que deriva en admissió de rols i permisos.
- Funcions de cerca basades en el motor Solr_. Indexació completament configurable, filtratge i autocompletar.
- Compatible amb OAI-PMH per tal de permetre la connectivitat de les dades (LinkedData) amb les institucions de coneixement.

L'aplicació inicial d'Arts Combinatòries és la d'allotjar l'arxiu cultural de la Fundació Antoni Tàpies i permetre'n el seu accés públic.

.. _Solr: http://lucene.apache.org/solr/

Elements tècnics
---------------------

Aquest projecte es composa de diferens components que han de ser instal·lats (veure Instal·lació) per a començar. A continuació es descriuen:

- Tomcat 7: Allotja l'aplicació bàsica que implemente els serveis REST, el Solr, i els serveis de vídeo.
- Plone 4: Capa de visió que actua com a client dels serveis REST i genera els continguts HTML.
- Openlink Virtuoso: Base de Dades de tripletes RDF per allotjar l'Ontologia, les dades RDF, processant SPARQL amb raonament.