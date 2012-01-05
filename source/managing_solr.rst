Managing Solr
======================================================================================

Solr is a search engine that supports full-text search of indexed documents, plus features for categorization and data filtering. Managing Solr properly is critical to provide easy user access to uploaded Ontology objects. Solr should not be directly dealed with, but through REST services that read from JSON Mapping specifications (already introduced in Configuration).

Clear
------------------

Clears all indexed data (commits automatically)

::

    Service path: http://{host:port}/{appname}/solr/clear
    HTTP Method: GET
    Returns: "success" or "error"

Commit
----------------

Commits recent indexation (not necessary)

::

    Service path: http://{host:port}/{appname}/solr/commit
    HTTP Method: GET
    Returns: media file

Schema
----------------

Generates schema from mapping JSON configuration.

::

    Service path: http://{host:port}/{appname}/solr/clear
    HTTP Method: GET
    Returns: "success" or "error"

NOTE: Solr must be restarted for new schema to take effect.

Reload
----------------

Peforms an update removing any existing data index. This process may take some time.

Solr indexation can be found at (SOLR_PATH)/data.xml

::

    Service path: http://{host:port}/{appname}/solr/reload
    HTTP Method: GET
    Returns: "success" or "error"

Update
----------------

Indexates new data according to mapping JSON configuration. This process may take some time.

Solr indexation can be found at (SOLR_PATH)/data.xml

This call is not recommended as there may remain indexed documents of unexisting objects in ontology. 

::

    Service path: http://{host:port}/{appname}/solr/update
    HTTP Method: GET
    Returns: "success" or "error"

Search
----------------

Peforms search, JSON result comes straight from Solr engine.

::

    Service path: http://{host:port}/{appname}/solr/search?s=Search&f=Filter&start=Start&rows=Rows&sort=Sort
    HTTP Method: GET
    Returns: Solr JSON result or "error"

"s" Search parameter, contains general text search.

"f" Filter parameter, defines results filtering in a "key:value" form sepparated by comas (see example below).

"start" and "rows" parameters define result scoping to provide pagination.

"sort" allows sorting results. Value must be field name to sort by, plus "desc" or "asc". Any used field here must have "sort" clause in mapping.json file (see Configuration)

**Search Example**

::

    http://internetdomain.org/ac/solr/search?s=James

**OK Result**

::

    {
        + "responseHeader": { ... },                    // Not relevant
        - "response":
            {
                "numFound": 277,
                "start": 0,
              - "docs": [
                  - {
                        "id": "James_Bond_ID"           // Note that search results consists in identifiers only, to get the full object data we use View services
                    },

                  - {
                        "id": "James_Stewart_ID"
                    },

                 -  {
                        "id": "James_Franco_ID"
                    },

                    // more results
                ]
            }

        - "facet_counts": {

             - "facet_fields": {                        // With adequate mapping we can get object categorization (see Configuration)
                    - "Birth": [
                        + "1901", "3",
                        + "1975", "2",
                        + "1930", "2",
                        // ...more
                      ]

                    - "Country": [
                        + "United States", "44",
                        + "United Kingdom", "33",
                        // ...more
                      ]
                }
            }
    }

**How filtering works**

Constructing filter parameter is quite simple. All we have to do is to chose what category value we want to filter from "facet_fields" section of JSON Solr result. Still in previous example, say we want to filter by Birth year and Country with specific values for each. Filter could be: "Birth:1930,Country:United Kingdom".

**Filtering Example**

::

    http://internetdomain.org/ac/solr/search?s=James&f=Birth:1930,Country:United+Kingdom   // Space characters can be replaced with "+"

**OK Result**

::

    {
        + "responseHeader": { ... },
        - "response":
            {
                "numFound": 1,
                "start": 0,
              - "docs": [
                  - {
                        "id": "James_Bond_ID"
                    }
                ]
            }

        - "facet_counts": {

             - "facet_fields": {                        
                    - "Birth": [
                        + "1930", "1",
                      ]

                    - "Country": [
                        + "United Kingdom", "1",
                      ]
                }
            }
    }

Note that you can filter by two values of the same category (for example: "Year:1930,Year:1975"). In this case the filter will be non-exclusive.

Autocomplete
------------------------

Performs autocomplete of a given search string, JSON result comes straight from Solr engine.

::

    Service path: http://{host:port}/{appname}/solr/autocomplete?s=Search
    HTTP Method: GET
    Returns: Solr JSON result or "error"

"s" Search parameter, contains general text search.

**Autocomplete Example**

::

    http://internetdomain.org/ac/solr/autocomplete?s=Ja

**OK Result**

::

    {
        + "responseHeader": { ... },                    // Not relevant
        + "response": { ... },                          // In autocomplete main response is neither relevant
        - "facet_counts": {

             - "facet_fields": {                       
                    - "Birth": [ ]
                    - "Country": [ 
                        + "Jamaica","1",
                        + "Japan","1",
                     ]
                    - "Person": [
                        + "Jack the ripper","1",
                        + "James Bond","1",
                        + "James Franco","1"
                        + "James Stewart","1"
                     ]
                }
            }
    }

Note that in autocomplete search, the faceted results comprise the fields marked as "autocomplete" in mapping.json (see Configuration).

Search configurations
-------------------------------

It is possible to customize search modes from server side. This is done by editing or creating the search.json in CONFIGURATIONS_PATH/mapping/ folder. Configuring searches allowes to add extra filtering to client request to focus its searches to a particular scope.
Say we want that, all searches focus only on Persons and Countries, ignoring other object types, and births from 1900 to 1950; the proper search.json would be as follows:

::
    
    {
        "name":"mycustomsearch"
        "type":"search",
        "value":["ObjectType:Person", "ObjectType:Country", "Birth:[1900 TO 1950]" ]
    },

    {
        "name":"default"
        "type":"search"
        // it can be left blank if we do not need additional filtering
    }

To use this search configuration, we add "config" parameter with value "mycustomsearch"

**Search Example**

::

    http://internetdomain.org/ac/solr/search?s=James&config=mycustomsearch

If "config" is not specified, "default" search configuration is used. If there's no such configuration or config value is not found in search.json, there's no additional filtering.

To get the available search configurations, you can use this service:

::

    Service path: http://{host:port}/{appname}/solr/configurations
    HTTP Method: GET
    Returns: Solr JSON result or "error"
