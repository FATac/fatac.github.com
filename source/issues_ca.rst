Notes de manteniment
==================================

Ontologies duplicades
-------------------------------

Després de cridar el servei "reset", totes les ontologies especificades a ONTOLOGY_NAMESPACES (config.json) es carreguen de nou. Com que han d'acabar amb el caràcter "#", el Virtuoso les carregarà dues vegades: una pel namespace acabat en "#", un altra pel namespace sense acabar en "#". Això no afecta el comportament normal de l'aplicació, però pot ser molest.

Pots esborrar les ontologies duplicades accedint a Virtuoso Conductor, clic a la pestanya RDF -> Pestanya "Graph" i esborrar les ontologies duplicades que NO acabin amb "#" (Compte d'esborrar grafs que no són ontologies, com el graf de les dades). 

Canvi de URL del servidor
---------------------------------

Si la URL del servidor canvia per qualsevol motiu, cal efectuar alguns canvis:

- Anar a config.json a establir la variable MEDIA_URL amb la URL del nou host (reiniciar l'aplicació web).
- Substituir totes les referències existents a la URL anterior amb la nova. Per a fer-ho, utilitzar el servei "replaceUri":

::

    Ruta servei: http://{host:port}/{appname}/replaceUri?uriField=URI_FIELD&oldUri=OLD_URI&newUri=NEW_URI
    Mètode HTTP: GET
    Retorna: "success" o "error"
    
**Exemple**

::

	http://internetdomain.org/ac/replaceUri?uriField=**my:Uri**&oldUri=**www.oldhost.com**&newUri=**www.newhost.com**
	
**OK Result**

::

	"success"
	
- Posar la URL a Plone "fatac settings": /fatac/@@fatac_settings

Còpia de seguretat
----------------------------

Per tal d'efectuar una copia de seguretat completa, cal copiar els elements següents:

- Tots els directoris especificats a la configuració (les propietats acabades en "PATH").
- Base de dades Virtuosos. Cal cridar "checkpoint;" des de la pantalla "Interactive SQL" del Virtuoso Conductor. Els fitxers a copiar són: virtuoso.db i virtuoso.trx localitzats a (directori virtuoso)/var/lib/virtuoso/db/

Servei de manteniment
----------------------------

Servei que realitza les següents tasques de manteniment regular de l'aplicació:

- Esborrar els fitxers temporals que es creen durant els processo legals
- Indexació de les dades a Solr (és a dir crida el servei /solr/reload)
- Actualització de les dades per a OAI-PMH (només si la variable OAI_PATH estigui definida)

::

    Ruta servei: http://{host:port}/{appname}/maintenance
    Mètode HTTP: GET
    Retorna: "success" o "error"
    
La crida d'aquest servei en un servidor a producció hauria de ser periòdica, a l'hora de menys tràfic d'usuaris/peticions.


