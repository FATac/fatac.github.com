Visualització
======================================================================================

Arts Combinatòries suporta diverses funcions de visualització de les dades, així com patrons i miniatures. Aquesta característica utilitza molts dels conceptes explicats en seccions posteriors, a les quals es faran contínues referències.

Object view
------------------

Servei que genera una vista JSON de l'objecte ontològic basant-se en el seu patró corresponent. Aquest és el patró amb el nom de la classe de l'objecte, o d'alguna de les seves superclasses. Si no hi ha cap template que correspongui, el servei retorna error.

::

    Ruta servei: http://{host:port}/{appname}/resource/{identifier}/view?section=NOM SECCIO
    Mètode HTTP: GET
    Retorna: JSON data or "error"
    
"section" permet especificar quina secció del patró-vista volem obtenir. Se'n poden especificar diverses, separant-les per coma ",". Si no s'especifica cap secció, s'obtenen totes les seccions de la vista.    

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

La distribució visual de les seccions pot prendre dues formes, una quan no hi ha cap element "search":

.. image:: layout1.jpg

i una altra quan sí que hi ha element "search":

.. image:: layout2.jpg

Es poden afegir tantes noves seccions com calguin, ja que seran afegides al final de la fitxa, tal com es representa amb la secció "footer" a les imatges anteriors.
    
** Nota per a modificar literals Plone **

Els noms de les dades "name", cal que siguin traduïts a Plone per tal que es corresponguin amb l'idioma de l'usuari. Això vol dir accedir als fitxers .po que pengen del directori locales i modificar-los adequadament. Per fer efectius els canvis cal executar translations.sh i reiniciar el Plone.

Cerca dins del patró d'objecte
-----------------------------------------

Tal com s'ha explicat a la Configuració, un template pot tenir un tipus de dada específica anomenat "search". Indica que s'hauria de realitzar una cerca Solr per tal d'omplir aquesta secció del patró. El motiu d'aquesta funcionalitat, és que alguns objectes poden referenciar un conjunt ampli d'objectes, i s'ha de permetre realitzar cerces sobre aquest conjunt.

Hi ha dues possibles aproximacions que porten al mateix resultat.

La **primera aproximació**, es pot descriure com a: des del conjunt d'objectes relacionats a un objecte específic. El que fem és mapejar (a mapping.json) el criteri del filtre que serà usat per a focalitzar els resultats a aquest conjunt d'objectes. Per exemple:

(mapping.json)
::

    {
        "data":
	    [
            /* .. resta del json */

            {
                "name":"RelatedPeople",                  
                "type":"string",                
                "path":["my:Person.my:knows"]      
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
                        "path:["my:Person.id"],
                        "value":["RelatedPeople:"],
                        "categories":["Year", "Location"]
				    }
			    ]
		    }
	    ]
    }

Després de crear la vista de l'objecte "James Bond", resulta en el següent:

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

    http://internetdomain.org/ac/solr/search?f=RelatedPeople:James_Bond_ID

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

A partir d'aquest domini inicial ("RelatedPeople:James_Bond_ID") es podrà procedir a afinar les cerques mitjançant els filtres. 

La **segona aproximació** té el sentit invers: des de l'objecte específic a tots els altres objectes relacionats. Aquí no hi ha mapeig adicional, i el patró és: 

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
                        "path:["my:Person.my:knows"],
                        "value":["id:"],                    // 'id' és un camp indexat per defecte, i és l'identificador de cada objecte
                        "categories":["Year", "Location"]
				    }
			    ]
		    }
	    ]
    }

El qual, després del realitzar la vista per l'objecte "James Bond", queda de la següent manera:

::


    {
	        /* ... resta del json */

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

El valor de la cerca ha de ser traduït a una crida a Solr (que portarà als mateixos resultats que l'aproximació anterior):

::

    http://internetdomain.org/ac/solr/search?f=id:M_Id,id:Q_Id,id:Miss_Moneypenny_ID,id:Dr_No_ID

Val la pena mencionar un darrer element, la llista de "categories" que descriu quines de les categories disponibles descrite al mapping.json s'adequen a aquesta vista. 

Miniatura d'objecte
----------------------------

Cridant el servei "thumbnail" es pot obtenir una imatge autogenerada que representi l'objecte.

::

    Ruta servei: http://{host:port}/{appname}/resource/{identifier}/thumbnail
    Mètode HTTP: GET
    Retorna: imatge jpg

Les mides de la miniatura s'especifiquen a la Configuració.

La generació de la imatge buscarà primer qualsevol imatge relacionada amb l'objecte. Això es fa resolent la dada tipus "objects" de la vista-patró de l'objecte. Altrament intenta el mateix amb la dada tipus "media" d'aquest patró. Si no hi ha patró o no es pot trobar cap d'aquestes dades, genera una miniatura genèrica segons la classe de l'objecte. Aquesta miniatura genèrica ha de situar-se  a (MEDIA_PATH)/thumbnails/classes com a (Nom-classe).jpg, on el nom de la classe pot ser el de l'objecte o el d'una de les seves superclasses. Com a últim recurs, la generació de miniatures recórre l'arxiu "default.jpg" del directori mencionat.  

La generació de miniatures és recursiva per objectes, el que implica que objectes relacionats amb altres objectes seran una composició de les miniatures dels objectes relacionats.

És possible a més, personalitzar la generació de la miniatura. Si es crea al patró-vista una secció anomenada "thumbnail" les dades d'aquesta secció seran exclusivament usades per a la generació de la miniatura.

**Exemple**

Suposem el patró de la classe "Country" que està composta per objectes "Location" i "Person":

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
					    "path":["my:Country.my:name"]
				    },

                    /* ... més dades */
			    ]
		    },
		
		    {
			    "name":"body",
			    "data":[
			
			     	{
			            "name":"Locations",
			            "type":"objects",
			            "path":["my:Country.my:hasLocation"]       // 'hasLocator' és una relació, per tant la ruta resol sempre a un identificador
			        },
			        
			        {
			            "name":"RellevantPeople",
			            "type":"objects",
			            "path":["my:Country.my:hasPerson"]       // 'hasPerson' és una relació, per tant la ruta resol sempre a un identificador
			        }
			    ]  
		    },
		    
		    {
			    "name":"thumbnail",						// L'existència d'aquesta secció "thumbnail" invalida la resta de seccions alhora de generar la miniatura
			    "data":[								// En aquest exemple es fa necessari repetir la dada "Locations" perquè volem generar la miniatura només a partir dels llocs del país
			
			     	{
			            "name":"Locations",
			            "type":"objects",
			            "path":["my:Country.my:hasLocation"]       
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
					    "path":["my:Location.my:name"]
				    },

                    /* ... més dades */
			    ]
		    },
		
		    {
			    "name":"body",
			    "data":[
			
			     	{
			            "name":"Representation",
			            "type":"media",
			            "path":["my:Location.my:imageUrl"]       // El tipus "media" hauria de resultar en una URL que contingui el media que podrà ser usat per generar la miniatura (una imatge o vídeo)
			        }
			    ]  
		    }
	    ]
    }
    
Fixem-nos que al patró "Location" no hi ha una secció "thumbnail", per tant es podran utilitzar totes les seccions per a generar la miniatura.

Una possible vista generada per l'objecte "Regne Unit" podria ser així:

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
					    "value":["Regne Unit"]
				    },

                    /* ... més dades */
			    ]
		    },
		    
		    {
			    "name":"body",
			    "data":[ ... ]  
		    },
		
		    {
			    "name":"thumbnail",
			    "data":[
			
			     	{
			            "name":"Locations",
			            "type":"objects",
			            "value":["London", "Birmingham", "Glasgow", "Liverpool", "Leeds", "Sheffield"]       // Semblen noms però són de fet identificadors
			        }
			    ]  
		    }
	    ]
    }

La generació de "London"...

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
					    "value":["Londres"]
				    },

                    /* ... més dades */
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

Amb la crida al servei

::

    http://internetdomain.org/ac/resource/United_Kingdom/thumbnail

les miniatures dels llocs (Locations) seran generats primer, accedint al seu "media" (si hi ha més d'una url, com a l'exmeple, es fa una composició). A continuació, la miniatura del país (Country) serà generada com una composició de les miniatures dels llocs relacionats.

Si hi ha tant dades tipus "media" com "objects" al patró, el "media" tenen prioritat perquè s'entén que tenen una relació més directa amb l'objecte.  

Totes les miniatures generades es desen al directori (MEDIA_PATH)/thumbnails per evitar haver-los de regenerar cada cop que es sol·liciten. Si cal que siguin regenerats, caldrà esborrar la imatge de la seva miniatura corresponent.


Class thumbnail
-----------------------

Servei que retorna la miniatura de la classe especificada. S'obté de la seva imatge corresponent ubicada al directori (MEDIA_PATH)/thumbnails/classes.

::

    Ruta servei: http://{host:port}/{appname}/classes/{prefix:nomClasse}/thumbnail?style=ESTIL_OPCIONAL
    Mètode HTTP: GET
    Retorna: imatge jpg
    
Les imatges del directori mencionat han de ser nombrades com a (prefix:NomClasse).jpg.

Opcionalment es poden afegir imatges adicionals de diferents estils afegint .nomEstil després del nom de la classe al nom de l'arxiu (Per exemple "my:Person.jpg", més "my:Person.gray.jpg", "my:Person.small.jpg", etc.). I cridar el servei usant el paràmetre "style" amb en nom de l'estil assignat per valor.

** Exemple OK **

::

	http://{host:port}/{appname}/classes/my:Person/thumbnail?style=gray		// retorna thumbnail d'estil "gray"
	http://{host:port}/{appname}/classes/my:Person/thumbnail				// retorna thumbnail sense estil
	

Llista i arbre de classes
----------------------------

Servei que retorna l'arbre de classes JSON

::

    Ruta servei: http://{host:port}/{appname}/classes/tree?c=CLASSE ARREL
    Mètode HTTP: GET
    Retorna: Arbre JSON o "error"

Servei que retorna la llista de classes JSON

::

    Ruta servei: http://{host:port}/{appname}/classes/list?c=CLASSE ARREL
    Mètode HTTP: GET
    Retorna: Llista JSON o "error"
