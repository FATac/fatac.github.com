OAI-PMH Support
======================================================================================

**OAI-PMH** (Open Archive Initiative - Protocol for Metadata Hervesting) standarizes archive data transmission between organizations. Arts Combinatòries allows to configure XML data export to make them avialable to OAICAT_ module, which must be incorporated.   

.. _OAICat http://www.oclc.org/research/activities/oaicat/default.htm

To have our data available in OAI-PMH, following steps must be considered:

Configure
----------------

A proper configuration of main application properties is required (Configuration Sep 3).

Also, OAI_PATH property must be set with directory path where XML files will be used by OAICat.

OAICat module must be configured the next way:
 
 1. In both files WEB-INF/oaicat.properties and WEB-INF/filesys.properties 
 2. Put in **FileSystemOAICatalog.homeDir** the directory path specified at OAI_PATH property
 3. Put in **FileMap2oai_dc.xsltName** the absolute path to XSL file WEB-INF/etdms2dc.xsl (or any other customized XSL)
 4. It might be necessary to configure <context-param><param-name>properties (...) of web.xml, setting in <param-value> the absolute path to oaicat.properties file

Mapping
------------

"oai-mapping.json" file must be added in (CONFIG_PATH)/mapping/ with a sort of content:  

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
	
Note that it's a mapping language equivalent to Solr indexation's. **xmlHeader**, **xmlFooter**, **xmlPrefix** depend on XSL that is used. Terms correspond to Dublin Core (DC) specification: title, creator, description, subject, date, type, identifier, etc.

Export
----------------

In order to having our data exported according to our mapping, next service should be used:

::

    Service path: http://{host:port}/{appname}/oai
    HTTP Method: GET
    Returns: "success" or "error"
    
This service call will lead to having a set of XML located in OAI_PATH 

OAICat
----------------

Upon file generation, our data is fully available for being harvested by third parties (for instance: http://host:port/oaicat) 