Suport per OAI-PMH
======================================================================================

**OAI-PMH** (Open Archive Initiative - Protocol for Metadata Hervesting) estandaritza la transmissió de dades d'arxiu entre institucions. Arts Combinatòries, permet configurar la transformació de les dades a XML per fer-les disponibles pel mòdul OAICat_, que s'ha d'incorporar al projecte. 

.. _OAICat http://www.oclc.org/research/activities/oaicat/default.htm

Els passos per a poder transmetre les pròpies dades en OAI-PHM són els següents:

Configurar
----------------

Cal haver configurat adeqüadament les propietats principals de l'aplicació (Pas 3r de la Configuració).

A més, la variable OAI_PATH ha d'indicar la ruta del directori on s'exportaran els fitxers XML que seran consumits per OAICat.

Cal configurar el módul OAICat de la següet manera:
 
 1. Arxiu WEB-INF/oaicat.properties
 2. Configurar la propietat **FileSystemOAICatalog.homeDir** amb la ruta especificada a OAI_PATH
 3. Configurar la propietat **FileMap2oai_dc.xsltName** amb la ruta absoluta al fitxer XSL de WEB-INF/etdms2dc.xsl (o a qualsevol XSL personalitzat)
 4. Podria ser necessari configurar el <context-param><param-name>properties (...) del web.xml, establint a <param-value> la ruta absoluta al fitxer oaicat.properties (p.ex: "/home/.../tomcat/webapps/WEB-INF/oaicat.properties")
 
Hi ha maneres adicionals d'utilitzar l'OAICat. La present explicació només mostra la configuració mínima necessària.

Mapeig
------------

Cal afegir a (CONFIG_PATH)/mapping/ l'arxiu "oai-mapping.json", amb un contingut com el següent:

::

	{
		"xmlHeader":"<oai_etdms:thesis xmlns:oai_etdms=\"http://www.ndltd.org/standards/metadata/etdms/1.0/\" xmlns:xsi=\"http://www.w3.org/2001/XMLSchema-instance\" xsi:schemaLocation=\"http://www.ndltd.org/standards/metadata/etdms/1.0/ http://www.ndltd.org/standards/metadata/etdms/1.0/etdms.xsd\">",
		"xmlFooter":"</oai_etdms:thesis>",
		"xmlPrefix":"oai_etdms",
	
		"data":
		[
			{
				"name":"title",
				"path":["my:Document.my:Title"]
			},
		
			{
				"name":"description",
			    "path":["my:Document.my:Description"]
			},
			
			{
				"name":"language",
				"path":["my:Document.my:hasTranslation=my:Language.my:Name"]
			}
			
			// ...
		]
	}
 
Com es pot comprovar, es tracta d'un llenguatge de mapeig equivalent al d'indexació a Solr. **xmlHeader**, **xmlFooter**, **xmlPrefix** depenen de l'XSL utilitzat. Els termes corresponen a l'especificació de Dublin Core (DC): title, creator, description, subject, date, type, identifier, etc.

Exportació
----------------

Per exportar les dades d'acord amb el mapeig anterior, cal utilitzar el servei següent:

::

    Ruta servei: http://{host:port}/{appname}/oai
    Mètode HTTP: GET
    Retorna: "success" o "error"
    
El resultat d'aquest crida serà un conjunt d'arxius XML localitzats a OAI_PATH

OAICat
----------------

Un cop els fitxers són generats, les nostres dades ja són disponibles per a ser collides per tercers (per exemple: http://host:port/oaicat)

  