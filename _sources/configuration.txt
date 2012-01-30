.. FAT Arts Combinatòries documentation master file, created by
   sphinx-quickstart on Tue May 31 12:39:26 2011.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

Configuration
======================================================================================

There are several aspects that must be defined in order to get Arts Combinatòries (AC) working with a new Ontology. This configuration guide is organized step-by-step.

Step 1: Virtuoso
---------------------------

Some Virtuoso initial configuration is necessary. Access to Virtuoso Conductor console (default should be: http://localhost:8890/conductor/ with both user and password "dba"). Click "Interactive SQL" on the right, and run the next sql script:

::

    -- Base script which creates necessary data structure for managing users, rights and uploaded media and objects
    -- Suitable (only?) for Openlink Virtuoso sql implementation

    -- DROP TABLE db.dba._media;
    -- DROP TABLE db.dba._thumbnail;
    -- DROP TABLE db.dba._right;
    -- DROP TABLE db.dba._identifier_counter;
    -- DROP TABLE db.dba._autodata;
    -- DROP TABLE db.dba._resource_statistics;

    CREATE TABLE db.dba._resource_statistics (
	    identifier VARCHAR(150),
	    visitCounter BIGINT,
	    creationMoment BIGINT,
	    lastMoment BIGINT
    )

    CREATE TABLE db.dba._media (
	    SID INT IDENTITY,
	    mediaId VARCHAR(60),
	    path VARCHAR(500),
	    moment TIMESTAMP
    )

    CREATE TABLE db.dba._thumbnail (
	    SID INT IDENTITY,
	    objectId VARCHAR(60),
	    path VARCHAR(500),
	    moment TIMESTAMP
    )

    CREATE TABLE db.dba._right (
	    SID INT IDENTITY,
	    objectId VARCHAR(60),
	    rightLevel INT
    )

    CREATE TABLE db.dba._identifier_counter (
	    identifier VARCHAR(150),
	    counter INT
    )

    CREATE TABLE db.dba._autodata (
	    keyName VARCHAR(100),
	    keyValue VARCHAR(100),
    	name VARCHAR(100),
	    defaultValue VARCHAR(100)
    )

    GRANT EXECUTE ON DB.DBA.SPARQL_DELETE_DICT_CONTENT TO "SPARQL";
    GRANT ALL PRIVILEGES TO "SPARQL";

(Note that you can uncomment "drops" to reset all tables)

Next, click "RDF" tab, and click "Namespaces", you must add here all linked ontology namespaces not included in the list. Keep in mind that those namespaces are eligible for use, and that all ontologies specified in "ONTOLOGY_NAMESPACES" in Step 3 must appear in this list.

Step 2: Solr
---------------------------

Upon Solr installation, a specific schema.xml must be used. This is generated according to mapping specifications (see Step 6) and to a base schema. This base schema is a file placed in "solr" directory (referenced in "SOLR_PATH" at Step 3), containing exactly the next content:

::

    <?xml version="1.0" encoding="UTF-8" ?>
    <!-- DO NOT MODIFY THIS FILE -->

    <schema name="mySchema" version="1.4">

     <types>
        <fieldType name="identifier" class="solr.StrField" sortMissingLast="true" omitNorms="true"/>
        <fieldType name="boolean" class="solr.BoolField" sortMissingLast="true" omitNorms="true"/>
        <fieldType name="long" class="solr.TrieLongField" precisionStep="0" omitNorms="true" positionIncrementGap="0"/> 
        <fieldType name="float" class="solr.TrieFloatField" precisionStep="0" omitNorms="true" positionIncrementGap="0"/>
        <fieldType name="int" class="solr.TrieIntField" precisionStep="0" omitNorms="true" positionIncrementGap="0"/> 

        <fieldType name="string" class="solr.TextField" sortMissingLast="true" omitNorms="true">
          <analyzer type="index">
            <tokenizer class="solr.KeywordTokenizerFactory"/>
            <charFilter class="solr.MappingCharFilterFactory" mapping="mapping-ISOLatin1Accent.txt"/>
            <charFilter class="solr.HTMLStripCharFilterFactory"/>
          </analyzer>
          <analyzer type="query">
            <tokenizer class="solr.KeywordTokenizerFactory"/>
            <charFilter class="solr.MappingCharFilterFactory" mapping="mapping-ISOLatin1Accent.txt"/>
            <charFilter class="solr.HTMLStripCharFilterFactory"/>
          </analyzer>
        </fieldType>

        <fieldType name="text_general" class="solr.TextField" positionIncrementGap="100">
          <analyzer type="index">
            <tokenizer class="solr.WhitespaceTokenizerFactory"/>
            <filter class="solr.WordDelimiterFilterFactory"
                            generateWordParts="1" generateNumberParts="1"
                            catenateWords="1" catenateNumbers="1"
                            catenateAll="0" preserveOriginal="1" />
            <filter class="solr.StopFilterFactory" words="stopwords.txt" ignoreCase="true"/>
            <filter class="solr.LowerCaseFilterFactory" />
            <filter class="solr.PatternReplaceFilterFactory" pattern="^(\p{Punct}*)(.*?)(\p{Punct}*)$" replacement="$2"/>
            <charFilter class="solr.MappingCharFilterFactory" mapping="mapping-ISOLatin1Accent.txt"/>
            <charFilter class="solr.HTMLStripCharFilterFactory"/>
          </analyzer>
          <analyzer type="query">
            <tokenizer class="solr.WhitespaceTokenizerFactory"/>
            <filter class="solr.WordDelimiterFilterFactory"
                            generateWordParts="1" generateNumberParts="1"
                            catenateWords="0" catenateNumbers="0"
                            catenateAll="0" preserveOriginal="1" />
            <filter class="solr.StopFilterFactory" words="stopwords.txt" ignoreCase="true"/>
            <filter class="solr.LowerCaseFilterFactory" />
            <filter class="solr.PatternReplaceFilterFactory" pattern="^(\p{Punct}*)(.*?)(\p{Punct}*)$" replacement="$2"/>
            <charFilter class="solr.MappingCharFilterFactory" mapping="mapping-ISOLatin1Accent.txt"/>
            <charFilter class="solr.HTMLStripCharFilterFactory"/>
          </analyzer>
        </fieldType>
     </types>

    <!-- FIELDS_INSERTION_MARK -->

     <uniqueKey>id</uniqueKey>

     <defaultSearchField>id</defaultSearchField>

     <solrQueryParser defaultOperator="OR"/>

    </schema>
	
Step 3: Main properties
----------------------------

The first thing we have to do is to configure the 'config.json' file, you may place them on your current directory. If you don't know which is the current dir you can see the AC log. Here's a sample with required properties and possible values: 

::

    {	
        "__comment_0":"Mixed config",

	    "THUMBNAIL_WIDTH":250,
	    "THUMBNAIL_HEIGHT":180,
	    "MEDIA_CONVERSION_PROFILES":["dv", "mpg", "avi", "aif", "mov"],
            "MEDIA_AUTOCONVERT":"false",
	    "LANGUAGE_LIST":["ca", "en", "es", "fr", "it", "de"],							
	    "USER_LEVEL":["*", "Member", "Manager+Reviewer", "Site Administrator"],	    
	
	    "__comment_1":"Services base URLs and connection strings",

	    "RDFDB_URL":"jdbc:virtuoso://myhost:1111",
	    "RDFDB_USER":"dba",
	    "RDFDB_PASS":"dba",
	    "REST_URL":"http://myhost:8080/rest/",
	    "SOLR_URL":"http://myhost:8080/solr/",
	    "VIDEO_SERVICES_URL":"http://myhost:8080/videoservices/rest/",
	    "USER_ROLE_SERVICE_URL":"http://myotherhost:8080/myapp/getUserRole?userId=",
	
        "__comment_2":"Ontology namespaces (After any change, all existing triples must be fixed)",

	    "RESOURCE_URI_NS":"http://localhost:8080/ArtsCombinatoriesRest/resource/",		
	    "RESOURCE_PREFIX":"ac_res",
	    "ONTOLOGY_NAMESPACES":[
		    "http://localhost:8080/rest/ontology/my#", "my",
		    "http://www.w3.org/1999/02/22-rdf-syntax-ns#", "rdf",
		    "http://www.w3.org/2000/01/rdf-schema#", "rdfs",
		    "http://dublincore.org/2010/10/11/dcterms.rdf#", "dcterms"
	    ],
	
	    "__comment_3":"Base directories that will be used by AC to allocate or access content and contiguratios",

	    "CONFIGURATIONS_PATH":"/achome/json/",
	    "SOLR_PATH":"/achome/solr/",
	    "MEDIA_PATH":"/achome/media/",
	    "ONTOLOGY_PATH":"/achome/myontology.owl",
	    "OAI_PATH":"..."
    }

THUMBNAIL_WIDTH and THUMBNAIL_HEIGHT determines the size of generated thumbnails.

MEDIA_CONVERSION_PROFILES enumerates video/audio file extensions suitable for conversion, ordered by profile number (e.g.: "dv" is profile 1, "mpg" is profile 2, etc.).

MEDIA_AUTOCONVERT set to "true" if you require that video/audo files to be converted once uploaded. Otherwise you can use "convert" service (see Managing Media section).

LANGUAGE_LIST enumerates codes of languages that are expected to be used in data base fields (the first one is used as default language).

USER_LEVEL specifies the degree of legal access that have each user role, ordered from more to less restrictions ("*" means any role). This list should contain only 4 elements as there are only 4 restriction levels. Each elements may contain more than one role, separated by '+' (p.ex: "Manager+Reviewer")

USER_ROLE_SERVICE_URL is a specific service url. This service is used by AC to resolve user groups, which will be considered to determine the permission acess of the user. Service must accept a user identifier (in the URL string) and should return one of the user groups specified in USER_LEVEL. 

ONTOLOGY_NAMESPACES establishes a prefix for each ontology/schema namespace, this prefix must also appear on namespaces list in Virtuoso (see Step 1). The first specified ontology must be the one specially created for this project (myOntology in the example), other specified ontologies/schemas must be the ones included on the first one. Generally, RDF and RDFS schemas should be always included.

AC requires the next folder and file structure in order to allocate and use its files:

- [CONFIGURATIONS_PATH]
    - legal/legal.json (required)
    - mapping/mapping.json (required)
    - mapping/search.json (optional)
    - mapping/ (optionally, json template definitons for any Ontology class named with prefix, example "foaf:Person.json")
- [SOLR_PATH] (Sorl home path)
    - conf/schema.xml-EMPTY (required)
    - data/data.xml (generated by application after indexing)
- [MEDIA_PATH]
    - thumbnail/
    - thumbnail/classes/default.jpg (required. Default thumbnail for all objects. Does not need to fit a specific size)
    - thumbnail/classes/ (optionally, default thumbnail for any classes Ontology class named with prefix, example "foaf:Person.jpg")
- [ONTOLOGY_PATH] (path to file containing the project's Ontology)

OAI_PATH is an optinal property explained in detail in OAI PMH Support section

Step 4: Reset
-----------------------------

Calling reset service, ALL data and media will be removed. Also last Ontology file (located in ONTOLOGY_PATH) will be loaded. 

::

    Service path: http://{host:port}/{appname}/reset?option=ontology&confirm=CURRENT_DATE
    HTTP Method: GET
    Returns: "success" or "error"

Set "option=ontology" if you do not want a total reset, but only a reload of all ontologies specified in ONTOLOGY_NAMESPACES.

Otherwise, for safety, "confirm" must be filled with current server date and time formated as "dd/mm/yy hh:mm"

**Examples**

::

    http://internetdomain.org/ac/reset?option=ontology               // ontologies reload

::

    http://internetdomain.org/ac/reset?confirm=11/11/2011 23:11      // data reset and ontologies reload



Step 5: Legal script
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

- Besides color result, license can also be assigned to the object. This is achieved by "license" clause. Its value can be as "my:hasLicense=License_ID", in other words: property name, "=" sign, and the ontologic object that corresponds to a license (for instance "Creative_Commons_ID"). This process does not check at all the consistency of the assigned license, this could even be any type of object. It corresponds to the user to seek the consistency of this process. 

Step 6: Data mapping
------------------------------

Data "mapping.json" (placed in json/mapping folder) is a must-have specification file that defines what ontology data must be indexed in Solr, and how this must be done. Data mapping is not a simple direct Owl to Solr mapping. It must be defined in a way that it later can be used for specific object domain searches (See Step 4), and provide additional information of the field nature to get Solr treating the data properly.

Let's say we have the Person class defined in our Ontology, and that we want to indexate several useful person data such as: name, biography, date of birth and birth place. Person indexation should be specified this way:

::

    {
	    "data":
	    [
            {
                "name":"Name",                      // Specifies the data identifier, in this case, the person Name
                "type":"string",                    // 'string' type means that values of Name will be treated as a whole
                "path":["my:Person.my:fullName"]    // Path to Class data property, note that it's specified as (Class-name).(property)
            },

            {
                "name":"Biography",             
                "type":"text",                  // 'text' makes every word (space separated tokens) to be treated separately on search
                "path":["my:Person.my:Bio"]           
            },

            {
                "name":"BirthDate",             
                "type":"date.year",             // 'date.year' will extract the year part of date value (default date format expected is dd/mm/yyyy)
                "path":["my:Person.my:BirthDate"]           
            },

            {
                "name":"BirthPlace",             
                "type":"string",                
                "path":["my:Person.my:BirthPlace=my:Location.my:Name"]   // Note that as Birth Place is not a string but an external object, specified path chains both objects, from original, to target data (Name property of Location class). You can chain as many objects as you need.
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
                "path":["my:Person.my:fullName", "my:Location.my:Name"]     // Path to Person and Location data property
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
                "path":["my:Person.my:fullName"],         
                "search":"yes",
                "autocomplete":"yes"
            },

            {
                "name":"Location",                                  
                "type":"string",                                
                "path":["my:Location.my:Name", "my:Person.my:BirthPlace=my:Location.my:Name"]
                "search":"yes",                     // Note that ALL clauses are unactive by default, 
                "autocomplete":"yes",               // so they must be always specified in case of need.
                "multilingual":"yes",
                "category":"yes"
            }

            /* rest of json ... */
        ]

Recommended practice:
- Group all text data -this is, all those that must be searched in word-by-word basis- inside the same data block, as "text" type.
- If it is required, also, categorize all results of some data already included in previous block, create a new block of type "string" and refer to the same data "path", and use clause "category:yes".
- If it is required, also, to use this data to sort results, create a new block of type "string" and again, refer to same data "path", and use clause "sort:yes"-
- About previous point, if sorting must be over different data of the same kind, include them all in the same block.

In short, it's recommended to sepparate data blocks by its function (search, category or sorting). Next mapping example shows all explained practices: 

::

    "data":
        [
            {
                "name":"Word_by_word_text_search",                                  
                "type":"text",                                
                "path":[
                	"my:Person.my:fullName",							// allows finding persons by name
                	"my:Person.my:biography",							// allows finding persons by bio
                	"my:Person.my:BirthPlace=my:Location.my:Name",		// allows finding persons by birthplace
                	"my:Location.my:Name",								// allows finding places by name
                	"my:Location.my:description"						// allows finding places by description
                ],         
                "search":"yes",
                "autocomplete":"yes"
            },

            {
                "name":"Place_category",                                  
                "type":"string",                                
                "path":[
                	"my:Location.my:Name",								// categorizes places
                	"my:Person.my:BirthPlace=my:Location.my:Name"		// categorizes persons birthplace
                ]
                "category":"yes"
            },
            
            {
                "name":"Year_sorting",                                  
                "type":"date.year",                                
                "path":[
                	"my:Person.my:BirthDate",							// allows sorting by birth year
                	"my:Location.my:FoundationDate"					// allows sorting by foundation year
                ]
                "sort":"yes"
            }

        ]

Step 7: Object template
------------------------------------

Any resource search will finally lead to individual object visualization. This makes it necessary to build templates for any Ontology object that should be visualizable. Object view is organized in sections, and each section contains a list of mapped data, in a similar way we used it in previous step.

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
					    "path":["my:Person.my:fullName"]
				    },

                    {
					    "name":"BirthDate",
					    "type":"date",
					    "path":["my:Person.my:BirthDate"]
				    },

                    {
					    "name":"BirthPlace",
					    "type":"linkedObject",
					    "path":["my:Person.my:BirthPlace=my:Location.my:Name"]
				    }
			    ]
		    },
		
		    {
			    "name":"body",
			    "data":[
			
			     	{
			            "name":"Biography",
			            "type":"text",
			            "path":["my:Person.my:Bio"]
			        }
			    ]  
		    },

            {
			    "name":"footer",                    
			    "data":[
		
		        	{
					    "name":"Related",
					    "type":"search",
                        "path":["my:Person.id"],
                        "value":["RelatedPeople:"],
                        "categories":["Year", "Location"]
				    }
			    ]
		    }
	    ]
    }


Data 'type' clause has not much to do with 'type' defined in previous step. The following types are all the ones available for templates:

- **text**: suitable for most cases, it resolves path to literal value with no modification.
- **linkedObject**: it shows resolved data path along with the referenced object id, separated by '@'. For example: London@my_london_id, this allows to create an hyperlink to the referenced object, which would be http://internetdomain.org/ac/resource/my_london_id/...
- **objects**: resolves path to identifier value.
- **media**: resolves path to media url value.
- **date**: and its parts (**date.year**, **date.day**, **date.month**). Same effect as date defined at step 3.
- **search**: this is a quite sophisticated object that comprises Solr searching feature from indexed data filtered by the specified constraint defined as combination of value and path. In this example: the search will only result to persons ("Person.knows:") that know current person ("Person.id"). For detailed information about searches please see Visualization page.
- **counter**: groups and counts same value results.

Please note that **text**, **objects** and **media** have the same effect. They resolve the path the same way but resulting value type is supposed to be different. See Visualization Object Thumbnail section to further in **media** and **objects** types.

Maintenance
----------------------------

Service that performs the next platform maintenance tasks:

- Delete all temporary files generated during legal process
- Index all data to Solr (this is a call to service /solr/reload)
- Updates data for OAI-PMH (if variable OAI_PATH is set)

::

    Service path: http://{host:port}/{appname}/maintenance
    HTTP Method: GET
    Returns: "success" or "error"
    
In a production server call to this service should be scheduled, called at the time when there is less traffic.
