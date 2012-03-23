Visualization
======================================================================================

AC supports several data visualization functions, such as templating and object thumbnail generation. This feature uses many concepts explained in previous sections, which will be constantly referred to.

Object view
------------------

REST service that generates Ontology object view based on its corresponding template. That is the template with name of current object class, or one of its superclasses. If there's no such template service returns error.

::

    Service path: http://{host:port}/{appname}/resource/{identifier}/view?section=SECTION NAME
    HTTP Method: GET
    Returns: JSON data or "error"
    
"section" parameter allows to select a specific section, or a few of them (sepparated by coma ","). If this parameter is not specified, all sections are returned. 

Template processing result of a Person (with template defined at Configuration) would be as follows:

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
					    "value":["United Kingdom"]
				    }
			    ]
		    },
		
		    {
			    "name":"body",
			    "data":[
			
			     	{
			            "name":"Biography",
			            "type":"text",
			            "value":["Secret agent in the service of the Queen"]
			        }
			    ]  
		    },

            {
			    "name":"content",                    
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


Sections layout can take two forms, when there is NO search element:

.. image:: layout1.jpg

and when there is a search element:

.. image:: layout2.jpg

Note that you can add as many sections as you want, as they will be added at the bottom, as represented by "footer" section in above images.

Search in object template
-------------------------------------

As described in Configuration Step 5, a template can have a specific object data type called "search". It indicates that Solr search should be performed in order to fill that part of the template with data. The aim of this feature is that some objects can relate to many other objects, and we want to be able to perform search and filtering of this particular scope. There are two possible approaches that lead to the same result.

The **first approach**, can be described as: from all other objects related to one specific object. What we do is to map (in mapping.json) the filtering criterion that will be followed to scope a particular object group. For example:

(mapping.json)
::

    {
        "data":
	    [
            /* .. rest of json*/

            {
                "name":"RelatedPeople",                  
                "type":"string",                
                "path":["my:Person.my:knows"]      
            }
        ]
    }

In the template we use this indexed field to get all related objects:

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
                        "path:["my:Person.id"],
                        "value":["RelatedPeople:"],
                        "categories":["Year", "Location"]
				    }
			    ]
		    }
	    ]
    }

Which after template process "James Bond" object results in following code:

::


    {
	        /* ... rest of json */

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

Since according to our mapping Solr indexes all related people for each Person, it makes sense to call Solr search filtering by "RelatedPeople" and value "James_Bond_ID" to get all related objects to this specific object.

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
                        "path:["my:Person.my:knows"],
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

    http://internetdomain.org/ac/solr/search?f=id:M_Id,id:Q_Id,id:Miss_Moneypenny_ID,id:Dr_No_ID

It's worth mentioning a last element of JSON template: "categories" list describe which of the available categories described in mapping.json are suitable for this scope.

Object thumbnail
----------------------------

Calling thumbnail service you can get an autogenerated image representing the object.

::

    Service path: http://{host:port}/{appname}/resource/{identifier}/thumbnail
    HTTP Method: GET
    Returns: jpg image

Thumbnail length and width can be Configured.

Image generation will first search for any related image to object. It's done by resolving any existing "objects" data type in object template, otherwise it tries to resolve any existing "media" data type in such template. If no template or none of those data types can be found in template, it generates a generic thumbnail according to the object class. This generic thumbnail must be placed in (MEDIA_PATH)/thumbnails/classes as (prefix:Class-name).jpg, where class-name can be the object's or one of it's superclasses. The last resort for thumbnail generation is "default.jpg" placed in mentioned directory. 

Thubmnail generation is object recursive, which means that thumbnails of objects related to other objects will be a composition of related objects thumbnails.

Still, thumbnail generation is customizable. If "thumbnail" section is created in the template, this section data will be exclusively used for thumbnail generation. 

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
					    "path":["my:Country.my:name"]
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
			            "path":["my:Country.my:hasLocation"]       // 'hasLocator' is a relation, so it resolves to an identifier
			        },
			        
			        {
			            "name":"RellevantPeople",
			            "type":"objects",
			            "path":["my:Cuntry.my:hasPerson"]       // 'hasPerson' is a relation, so it resolves to an identifier
			        }
			    ]  
		    },
		    
		    {
			    "name":"thumbnail",						// "thumbnail" section invalidates all any other section for thumbnail generation
			    "data":[								// In this example: it's necessary to repeat "Locations" data as we only need a thumbnail composed by the Country's locations. 
			
			     	{
			            "name":"Locations",
			            "type":"objects",
			            "path":["my:Cuntry.my:hasLocation"]       
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

                    /* ... more JSON data */
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
			            "name":"Representation",
			            "type":"media",
			            "path":["my:Location.my:imageUrl"]       // Media type should resolve to an URL containing any media (image, video, etc.)
			        }
			    ]  
		    }
	    ]
    }
    
Note that in Location template there is no "thumbnail" section, so any other section can be used for thumbnail generation.    

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
			    "data":[ ... ]  
		    },
		
		    {
			    "name":"thumbnail",
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

    http://internetdomain.org/ac/resource/United_Kingdom/thumbnail

thumbnails of related Location objects will be generated first, by accessing to their media (if there's more than one media url, as shown in example, an image composition is performed). Next, thumbnail of Country will be generated as a composition of related object thumbnails.

If there are both "media" and "objects" data types in the template, "media" references have priority for thumbnail generation for they are considered to be more reliable for object representation.

All generated thumbnails are saved in folder (MEDIA_PATH)/thumbnails to avoid regenerating on every access. If you need them to be regenerated, you have to remove their corresponding thumbnail image files.


Class thumbnail
-----------------------

Service that return the specified class thumbnail. This is obtained from its corresponding image located in (MEDIA_PATH)/thumbnails/classes folder.

::

    Service path: http://{host:port}/{appname}/classes/{prefix:className}/thumbnail?style=OPTIONAL_STYLE
    HTTP Method: GET
    Returns: jpg image
    
Image files in mentioned directory must be named as (prefix:ClassName).jpg. 

You can optionally add other image styles by adding .styleName after class name in file name (for instance "my:Person.jpg" plus "my:Person.gray.jpg", "my:Person.small.jpg", etc. ). And call the service using "style" parameter with the style name value assigned.

** OK Example **

::

	http://{host:port}/{appname}/classes/my:Person/thumbnail?style=gray		// return thumbnail of "gray" style
	http://{host:port}/{appname}/classes/my:Person/thumbnail				// return thumbnail of no style


Class list and tree
-----------------------

Service that returns Ontology classes tree in JSON

::

    Service path: http://{host:port}/{appname}/classes/tree?c=ROOT CLASS
    HTTP Method: GET
    Returns: JSON tree or "error"

Service that returns Ontology classes list in JSON

::

    Service path: http://{host:port}/{appname}/classes/list?c=ROOT CLASS
    HTTP Method: GET
    Returns: JSON list or "error"
