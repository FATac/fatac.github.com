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

Template processing result of a Person (with template defined at Configuration Step ) would be as follows:

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

As described in Configuration Step 4, a template can have a specific object data type called "search". It indicates that Solr search sould be performed in order to fill that part of the template with data. The aim of this feature is that some objects can relate to many other objects, and we want to be able to perform search and filtering of this particular scope.

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



