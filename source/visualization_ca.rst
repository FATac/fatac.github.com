Visualització
======================================================================================

Arts Combinatòries suporta diverses funcions de visualització de les dades, així com patrons i miniatures. Aquesta característica utilitza molts dels conceptes explicats en seccions posteriors, a les quals es faran contínues referències.

Object view
------------------

Servei que genera una vista JSON de l'objecte ontològic basant-se en el seu patró corresponent. Aquest és el patró amb el nom de la classe de l'objecte, o d'alguna de les seves superclasses. Si no hi ha cap template que correspongui, el servei retorna error.

::

    Ruta servei: http://{host:port}/{appname}/resource/{identifier}/view
    Mètode HTTP: GET
    Retorna: JSON data or "error"

Utilitzant el patró definit a la Configuració. Un objecte Person específic tindria la següent vista:

::


    {
	    "className":"Person",
	
	    "sections":
	    [
		    {
			    "name":"header",                    
			    "data":[
		
		        	{
					    "name":"Name",
					    "type":"text",
					    "value":["James Bond"]
				    },

                    {
					    "name":"BirthDate",
					    "type":"date",
					    "value":["01/01/1930"]
				    },

                    {
					    "name":"BirthPlace",
					    "type":"linkedObject",
					    "value":["Regne Unit"]
				    }
			    ]
		    },
		
		    {
			    "name":"body",
			    "data":[
			
			     	{
			            "name":"Biography",
			            "type":"text",
			            "value":["Agent secret al servei de la Reina"]
			        }
			    ]  
		    },

            {
			    "name":"footer",                    
			    "data":[
		
		        	{
					    "name":"Related",
					    "type":"search",
                        "value":["RelatedPeople:James_Bond_ID"],
                        "categories":["Year", "Location"]
				    }
			    ]
		    }
	    ]
    }

Cerca dins del patró d'objecte
-----------------------------------------

Tal com s'ha explicat a la Configuració, un template pot tenir un tipus de dada específica anomenat "search". Indica que s'hauria de realitzar una cerca Solr per tal d'omplir aquesta secció del patró. El motiu d'aquesta funcionalitat, és que alguns objectes poden referenciar un conjunt ampli d'objectes, i s'ha de permetre realitzar cerces sobre aquest conjunt.

Hi ha dues possibles aproximacions que porten al mateix resultat.

La **Primera aproximació**, es pot descriure com a: des del conjunt d'objectes relacionats a un objecte específic. El que fem és mapejar (a mapping.json) el criteri del filtre que serà usat per a focalitzar els resultats a aquest conjunt d'objectes. Per exemple:

(mapping.json)
::

    {
        "data":
	    [
            /* .. resta del json */

            {
                "name":"RelatedPeople",                  
                "type":"string",                
                "path":["Person.knows"]      
            }
        ]
    }


Al patró, usem aquest camp indexat per tal d'obtenir tots els objectes relacionats.

(Person.json)
::


    {
	        /* ... resta del json */

            {
			    "name":"footer",                    
			    "data":[
		
		        	{
					    "name":"Related",
					    "type":"search",
                        "path:["Person.id"],
                        "value":["RelatedPeople:"],
                        "categories":["Year", "Location"]
				    }
			    ]
		    }
	    ]
    }

Després de crear la vista de l'objecte "James Bons", resulta en el següent:

::


    {
	        /* ... resta del json */

            {
			    "name":"footer",                    
			    "data":[
		
		        	{
					    "name":"Related",
					    "type":"search",
                        "value":["RelatedPeople:James_Bond_ID"],
                        "categories":["Year", "Location"]
				    }
			    ]
		    }
	    ]
    }

Ja que d'acord amb el nostre mapeig, el Solr ha indexat totes les persones relacionades amb cada persona (Person.knows), té sentit cridar la cerca solr filtrant per "RelatedPeople" i valor "James_Bond_ID" per obtenir el conjunt d'objectes relacionats amb aquest.

::

    http://internetdomain.org/rest-path/solr/search?f=RelatedPeople:James_Bond_ID

**OK Result**

::

    {
        + "responseHeader": { ... },
        - "response":
            {
                "numFound": 4,
                "start": 0,
              - "docs": [
                  - {
                        "id": "M_Id"
                    },

                  - {
                        "id": "Q_Id"
                    },

                  - {
                        "id": "Miss_Moneypenny_ID"
                    },

                  - {
                        "id": "Dr_No_ID"
                    }
                ]
            }

        - "facet_counts": {

             - "facet_fields": {                        
                    - "Birth": [
                        + "1937", "1",
                        + "1925", "2",
                        + "1912", "1"
                      ]

                    - "Country": [
                        + "United Kingdom", "4",
                      ]
                }
            }
    }

Basing all our searches on this scope ("RelatedPeople:James_Bond_ID") we can perform more specific searches.

The **second approach** has the inverse description: from one specific object to all other related objects. There is no additional mapping, and template file 

(Person.json)
::


    {
	        /* ... rest of json */

            {
			    "name":"footer",                    
			    "data":[
		
		        	{
					    "name":"Related",
					    "type":"search",
                        "path:["Person.knows"],
                        "value":["id:"],                    // 'id' is a default indexated field and it is the identifier of every object
                        "categories":["Year", "Location"]
				    }
			    ]
		    }
	    ]
    }

Which after template process "James Bond" object results to following code:

::


    {
	        /* ... rest of json */

            {
			    "name":"footer",                    
			    "data":[
		
		        	{
					    "name":"Related",
					    "type":"search",
                        "value":["id:M_Id", "id:Q_Id", "id:Miss_Moneypenny_ID", "id:Dr_No_ID"],
                        "categories":["Year", "Location"]
				    }
			    ]
		    }
	    ]
    }

That must be traduced to Solr search call (that will lead to same results as previous approach):

::

    http://internetdomain.org/rest-path/solr/search?f=id:M_Id,id:Q_Id,id:Miss_Moneypenny_ID,id:Dr_No_ID

It's worth mentioning a last element of JSON template: "categories" list describe which of the available categories described in mapping.json are suitable for this scope.

Object thumbnail
----------------------------

Calling thumbnail service you can get an autogenerated image representing the object.

::

    Ruta servei: http://{host:port}/{appname}/resource/{identifier}/thumbnail
    Mètode HTTP: GET
    Retorna: jpg image

Thumbnail length and width can be Configured.

Image generation will first search for any related image to object. It's done by resolving any existing "objects" data type in object template, otherwise it tries to resolve any existing "media" data type in such template. If no template or none of those data types can be found in template, it generates a generic thumbnail according to the object class. This generic thumbnail must be placed in (MEDIA_PATH)/thumbnails/classes as (Class-name).jpg, where class-name can be the object's or one of it's superclasses. The last resort for thumbnail generation is "default.jpg" placed in mentioned directory. 

Thubmnail generation is object recursive, which means that thumbnails of objects related to other objects will be a composition of related objects thumbnails.

**Example**

Suppose template of a class Country that is composed of Locations

(Country.json)
::

    {
	    "className":"Country",
	
	    "sections":
	    [
		    {
			    "name":"header",                    
			    "data":[
		
		        	{
					    "name":"Name",
					    "type":"text",
					    "path":["Country.name"]
				    },

                    /* ... more JSON data */
			    ]
		    },
		
		    {
			    "name":"body",
			    "data":[
			
			     	{
			            "name":"Locations",
			            "type":"objects",
			            "path":["Cuntry.hasLocation"]       // Note that 'hasLocation' is an object property, so path resolves to identifier
			        }
			    ]  
		    }
	    ]
    }

(Location.json)
::

    {
	    "className":"Location",
	
	    "sections":
	    [
		    {
			    "name":"header",                    
			    "data":[
		
		        	{
					    "name":"Name",
					    "type":"text",
					    "path":["Location.name"]
				    },

                    /* ... more JSON data */
			    ]
		    },
		
		    {
			    "name":"body",
			    "data":[
			
			     	{
			            "name":"Representation",
			            "type":"media",
			            "path":["Location.imageUrl"]       // Media type should resolve to an URL containing any media (image, video, etc.)
			        }
			    ]  
		    }
	    ]
    }

Possible Country template resolution for object "United Kingdom" could be as follows

::

    {
	    "className":"Country",
	
	    "sections":
	    [
		    {
			    "name":"header",                    
			    "data":[
		
		        	{
					    "name":"Name",
					    "type":"text",
					    "value":["United Kingdom"]
				    },

                    /* ... more JSON data */
			    ]
		    },
		
		    {
			    "name":"body",
			    "data":[
			
			     	{
			            "name":"Locations",
			            "type":"objects",
			            "value":["London", "Birmingham", "Glasgow", "Liverpool", "Leeds", "Sheffield"]       // They look like names but they're ids in fact
			        }
			    ]  
		    }
	    ]
    }

And resolution of "London"...

::

    {
	    "className":"Location",
	
	    "sections":
	    [
		    {
			    "name":"header",                    
			    "data":[
		
		        	{
					    "name":"Name",
					    "type":"text",
					    "value":["London"]
				    },

                    /* ... more JSON data */
			    ]
		    },
		
		    {
			    "name":"body",
			    "data":[
			
			     	{
			            "name":"Representation",
			            "type":"media",
			            "value":["http://myhost/../mylondonImage1.jpg", 
                                "http://myhost/../mylondonImage2.jpg",
                                "http://myhost/../mylondonImage3.jpg", ... ]
			        }
			    ]  
		    }
	    ]
    }

Upon call to service

::

    http://internetdomain.org/rest-path/resource/United_Kingdom/thumbnail

thumbnails of related Location objects will be generated first, by accessing to their media (if there's more than one media url, as shown in example, there's an image composition). Next, thumbnail of Country will be generated as a composition of related object thumbnails.

If there are both "media" and "objects" data types in the template, "media" references have priority for thumbnail generation. 

All generated thumbnails are saved in folder (MEDIA_PATH)/thumbnails to avoid regenerating at every access. If you need them to be regenerated, you have to remove their corresponding thumbnail image files.


Class list and tree
-----------------------

Service that Retorna Ontology classes tree in JSON

::

    Ruta servei: http://{host:port}/{appname}/classes/tree?c=ROOT CLASS
    Mètode HTTP: GET
    Retorna: JSON tree or "error"

Service that Retorna Ontology classes list in JSON

::

    Ruta servei: http://{host:port}/{appname}/classes/list?c=PARENT CLASS
    Mètode HTTP: GET
    Retorna: JSON list or "error"
