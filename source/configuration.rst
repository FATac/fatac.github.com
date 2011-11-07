.. FAT Arts Combinatòries documentation master file, created by
   sphinx-quickstart on Tue May 31 12:39:26 2011.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

Configuration
======================================================================================

There are several aspects that must be defined in order to get the Semantic Web (SW) working with a new Ontology. This configuration guide is organized step-by-step.

Step 1: Main properties
----------------------------

Once installed (see Installation), the first thing we have to do is to configure the 'config.properties' file, you may place them on your current directory. Here's a sample with detailed explanations:

::

     # Core application supports thumbnail generation for any kind of object that may or may not contain data. Set thumbnail width and length in pixels.
	 THUMBNAIL_WIDTH = 250
	 THUMBNAIL_HEIGHT = 180
     
     # Video format extensions that are eligible for conversion to open format (OGV)
	 VIDEO_FILE_EXTENSIONS = dv,mpg,avi

     # Language code list that should be considered to be in ontology. 

     # Languages that are not in the list are ignored. The first language on the list is considered as default language
	 LANG_LIST = ca,en,es,fr,it,de

     # Services base URLs and connection strings
	 RDFDB_URL = jdbc:virtuoso://localhost:1111
	 RDFDB_USER = 
	 RDFDB_PASS = 

     # URL of REST services ideally reachable from any Internet location
	 REST_URL = http://internetdomain.org/rest-path/

     # URL of Solr search engine, local or not
	 SOLR_URL = http://localhost:8080/solr/

     # URL of Video conversion services, local or not
	 VIDEO_SERVICES_URL = http://localhost:8080/video-services/rest/

     # Own ontology and data resource namespaces. Notice that data and ontology is reparated for flexibility purposes
	 ONTOLOGY_URI_NS = http://internetdomain.org/rest-path/ontology/ac#
	 ONTOLOGY_PREFIX = ac
	 RESOURCE_URI_NS = http://internetdomain.org/rest-path/resource/	
	 RESOURCE_PREFIX = ac_res
	 RDF_URI_NS = http://www.w3.org/1999/02/22-rdf-syntax-ns#
	 RDFS_URI_NS = http://www.w3.org/2000/01/rdf-schema#

     # ATENTION: LinkedData best practices requires that RESOURCE_URI_NS, ONTOLOGY_URI_NS and REST_URL should have the same uri base (as specified in this sample) for REST services 'ontology' and 'resource' are implemented. 

     # Also, ONTOLOGY_URI_NS termination must coincide with ONTOLOGY_PREFIX plus '#' character
	
     # Json directory containing object templates and mapping specifications
	 CONFIGURATIONS_PATH = /var/ac-project/json/

     # Solr base directory to access schema. This folder must placed where Solr can reach it (usually in current directory)
	 SOLR_PATH = /mycurrentdirectory/solr/

     # Media directory where all uploaded media filles will be stored, it must contain 'thumbnail' subfolder, and 'classes' subsubfolder inside
	 MEDIA_PATH = ./var/ac-project/media/

     # Path to OWL file containing the main Ontology. This file will be uploaded on every call to 'reset' service.
	 ONTOLOGY_PATH = /var/ac-project/OntologiaArtsCombinatories.owl

Step 2: Legal script
-----------------------------

AC provides capabilities for assigning legal rights to media objects. The right assignation is an user assisted process that can be scripted and fully customized. (If you have no intention to apply this feature you may skip this step).

There is a self-explanatory sample named 'legal.json' in json directory, 'legal' subfolder. 'legal.json' is the name of the script file that will assist the user, the main parts of the script are:

- Start Block: starting block of the script
- Blocks: list of blocks the process will run through.
- Block name: name of block user for referencing it from other blocks
- Block description: additional explanation of block aim
- Block data: data that will be requested to user (as a user form) and will be used to resolve the right assignation. This data is considered global, so it can be reused or reassigned in further blocks.
- Block rules: data evaluation using boolean expressions. It can result to a next block, indicaded by 'block' keyword, or to a color indicated by 'result' keyword. Color consequences is explained next.

There are four "trafic light" colors that can be assigned to any object as a result of the legal process. From less to more restrictive: "green", "yellow", "orange" and "red". Each of one corresponding to one accessing right level from 1 to 4. On every call to a service that provides media data, the accessing level must be specified. Service will fail if user accessing level is lower than object restriction level. Eg. User level = 1 , Object level = 2 --> Fail / User level = 2 , Object level = 2 --> OK.

Step 3: Data mapping
------------------------------

Data "mapping.json" (placed in json/mapping folder) is a must-have specification file that defines what ontology data must be indexed in Solr, and how this must be done. Data mapping is not a simple direct Owl to Solr mapping. It must be defined in a way that it later can be used for specific object domain searches (See Step 4), and provide additional information of the field nature to get Solr treating the data properly.

Let's say we have the Person class defined in our Ontology, and that we want to indexate several useful person data such as: name, biography, date of birth and birth place. Person indexation should be specified this way:

::

    {
	    "data":
	    [
            {
                "name":"Name",                  // Specifies the data identifier, in this case, the person Name
                "type":"string",                // 'string' type means that values of Name will be treated as a whole
                "path":["Person.fullName"]      // Path to Class data property, note that it's specified as (Class-name).(property)
            },

            {
                "name":"Biography",             
                "type":"text",                  // 'text' makes every word (space separated tokens) to be treated separately on search
                "path":["Person.Bio"]           
            },

            {
                "name":"BirthDate",             
                "type":"date.year",             // 'date.year' will extract the year part of date value (default date format expected is dd/mm/yyyy)
                "path":["Person.BirthDate"]           
            },

            {
                "name":"BirthPlace",             
                "type":"string",                
                "path":["Person.BirthPlace:Location.Name"]   // Note that as Birth Place is not a string but an external object, specified path chains both objects, from original, to target data (Name property of Location class). You can chain as many objects as you need.
            }
        ]
    }

Note that path is a json array, this makes it possible to specify various object indexation. Let's suppose that we want to indexate one more object: Locations (with property Name). Code should be modified as follows:

::

    "data":
        [
            {
                "name":"ObjectClass",           // This is not mandatory but totally recommended: As we have now more than one object type, 
                                                // specifying this data, will allow filtering searches by object class.
                "type":"string",
                "path":["*.class"]              // We want no specific class by '*' character instead of class name, 
                                                // and we use reserved word 'class' to get the indexated object class name. 
                                                // 'superclass', and 'id' are also a reserved words, with obvious results.
            },

            {
                "name":"Name",                                  
                "type":"string",                                
                "path":["Person.fullName", "Location.Name"]     // Path to Person and Location data property
            },

            /* rest of json ... */
        ]

To provide proper searches, we can specify additional clauses for each data:

- **category**: Solr searches will use 'facets' feature to categorize specified data values by grouping and counting equal matches.
- **multilingual**: Applicable to data introduced in various languages in RDF database. For instance, a person biography can be written in different languages. This prevents Solr search from returning the same data in different languages.
- **search**: This might sound obvious that all mapped data should be user for search, but it's not. There may be data that's interesting only as a search result but not for searching in its string value. Unless you specify this clause, mapped data is not considered for searching.
- **autocomplete**: Only if you specified the previous clause, you can activate autocomplete to get this data in the autocomplete search.

For example: 'Name' data (that is, person and location name) is interesting for search and autocomplete. But Person name is specified in single language, and Location name is specified in different languages. Also, we find interesting to categorize results by locations but not by persons. According to all this, previous json code should change as follows:

::

    "data":
        [
            {
                "name":"Person",                                  
                "type":"string",                                
                "path":["Person.fullName"],         
                "search":"yes",
                "autocomplete":"yes"
            },

            {
                "name":"Location",                                  
                "type":"string",                                
                "path":["Location.Name", "Person.BirthPlace:Location.Name"]
                "search":"yes",                     // Note that ALL clauses are unactive by default, 
                "autocomplete":"yes",               // so they must be always specified in case of need.
                "multilingual":"yes",
                "category":"yes"
            }

            /* rest of json ... */
        ]


Step 4: Object template
------------------------------------

Any object search will finally lead to individual object visualization. This makes it necessary to build templates for any Ontology object that should be visualizable. Object view is organized in sections, and each section contains a list of mapped data, in a similar way we used it in previous step.

Going back to Person object class example: name, birth date, and birth place should be placed at header. Biography can be placed at body, we can also use a 'knows' relation to get related Persons and we can place this at footer section. (Note that sections are totally customizable).

The resulting template file must be placed as "Person.json" (generally, (Class-name).json) in json/mapping directory. Code should look as follows:

::

    {
	    "className":"Person",
	
	    "sections":
	    [
		    {
			    "name":"header",                    // section name
			    "data":[
		
		        	{
					    "name":"Name",
					    "type":"text",
					    "path":["Person.fullName"]
				    },

                    {
					    "name":"BirthDate",
					    "type":"date",
					    "path":["Person.BirthDate"]
				    },

                    {
					    "name":"BirthPlace",
					    "type":"linkedObject",
					    "path":["Person.BirthPlace:Location.Name"]
				    }
			    ]
		    },
		
		    {
			    "name":"body",
			    "data":[
			
			     	{
			            "name":"Biography",
			            "type":"text",
			            "path":["Person.Bio"]
			        }
			    ]  
		    },

            {
			    "name":"footer",                    
			    "data":[
		
		        	{
					    "name":"Related",
					    "type":"search",
                        "path":["Person.id"],
                        "value":["RelatedPeople:"],
                        "categories":["Year", "Location"]
				    }
			    ]
		    }
	    ]
    }


Data 'type' clause has not much to do with 'type' defined in previous step. Previous template example we use all data types available for templates:

- **text**: suitable for most cases, it shows data as it's resolved with no modification.
- **linkedObject**: it shows resolved data path along with the referenced object id, separated by '@'. For example: London@my_london_id, this allows to create an hyperlink to the referenced object, which would be http://internetdomain.org/rest-path/resource/my_london_id/...
- **date**: and its parts (**date.year**, **date.day**, **date.month**). Same effect as date defined at step 3.
- **search**: this is a quite sophisticated object that comprises Solr searching feature from indexed data filtered by the specified constraint defined as combination of value and path. In this example: the search will only result to persons ("Person.knows:") that know current person ("Person.id"). For detailed information about searches please see Visualization page.

