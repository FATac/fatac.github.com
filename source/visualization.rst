Visualization
======================================================================================

AC supports enhanced data visualization functions, such as templating and object thumbnail generation. This feature uses many of the principles explained in previous sections, which will be constantly referred to.

Object view
------------------

REST service that generates Ontology object view based on its corresponding template. That is the template with name of current object class, or one of its superclasses. If there's no such template service returns error.

::

    Service path: http://{host:port}/{appname}/resource/{identifier}/view
    HTTP Method: GET
    Returns: JSON data or "error"

Template processing result of a Person (with template defined at Configuration 5) would be as follows:

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

Search in object template
-------------------------------------

As described in Configuration Step 5, a template can have a specific object data type called "search". It indicates that Solr search sould be performed in order to fill that part of the template with data. The aim of this feature is that some objects can relate to many other objects, and we want to be able to perform search and filtering of this particular scope. There are to possible approaches that lead to same result.

The **first approach**, can be described as: from all other objects related to one specific object. What we do is to map (in mapping.json) the filtering criteria that will be used to scope a particular object group. For example:

(mapping.json)
::

    {
        "data":
	    [
            /* .. rest of json*/

            {
                "name":"RelatedPeople",                  
                "type":"string",                
                "path":["Person.knows"]      
            }
        ]
    }

In the template we use this criteria to get all related objects:

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
                        "path:["Person.id"],
                        "value":["RelatedPeople:"],
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
                        "value":["RelatedPeople:James_Bond_ID"],
                        "categories":["Year", "Location"]
				    }
			    ]
		    }
	    ]
    }

Since according to our mapping Solr indexated all related people for each Person, it makes sense to call Solr search filtering by "RelatedPeople" and value "James_Bond_ID" to get all related objects to this specific object.

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

    Service path: http://{host:port}/{appname}/resource/{identifier}/thumbnail
    HTTP Method: GET
    Returns: jpg image

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

Service that returns class tree in JSON

::

    Service path: http://{host:port}/{appname}/classes/tree?c=ROOT CLASS
    HTTP Method: GET
    Returns: JSON tree or "error"

Service that returns class list in JSON

::

    Service path: http://{host:port}/{appname}/classes/list?c=PARENT CLASS
    HTTP Method: GET
    Returns: JSON list or "error"





