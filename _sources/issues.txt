Maintenance notes
==================================

Ontology duplication
-------------------------------

Upon calling "reset" service, all specified ontologies in ONTOLOGY_NAMESPACES (config.json) are reloaded. As they have to end with "#" character, Virtuoso will load them twice, one for the ontology namespace ended with "#" and another one for the namespace without this ending character. This does not affect the normal behaviour of the application, but it can be annoying. 

You can delete duplicate ontologies by acessing to Virtuoso Conductor, click RDF tab -> Graphs tab -> And delete the loaded ontologies that does NOT end with "#" (Beware of deleting other non-ontology graphs such as data graph).

Server URL change
-------------------------------

If server URL is changed by any reason, some changes must be performed:

- Go to config.json and set MEDIA_URL with the new host URL (restart web application).
- Replace all existing references from the previous host URL to the new one. To do so, use "replaceUri" service:

::

    Service path: http://{host:port}/{appname}/replaceUri?uriField=URI_FIELD&oldUri=OLD_URI&newUri=NEW_URI
    HTTP Method: GET
    Returns: "success" or "error"
    
**Example**

::

	http://internetdomain.org/ac/replaceUri?uriField=**my:Uri**&oldUri=**www.oldhost.com**&newUri=**www.newhost.com**
	
**OK Result**

::

	"success"
	
- Set new URL at Plone fatac settings: /fatac/@@fatac_settings

Data backup
----------------------------

In order to perform a complete data backup, next elements are necessary to be safely copied:

- All directories specified in configuration ("PATH" ended properties).
- Virtuoso database. "checkpoint;" must be called in Virtuoso conductor "Interactive SQL". The files to be copied are: virtuoso.db and virtuoso.trx located in (virtuoso directory)/var/lib/virtuoso/db/

Maintenance service
-------------------------------

Service that performs the next platform maintenance tasks:

- Delete all temporary files generated during legal process
- Index all data to Solr (this is a call to service /solr/reload)
- Updates data for OAI-PMH (if variable OAI_PATH is set)

::

    Service path: http://{host:port}/{appname}/maintenance
    HTTP Method: GET
    Returns: "success" or "error"
    
In a production server call to this service should be scheduled, called at the time when there is less traffic.
