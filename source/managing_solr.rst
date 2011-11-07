Managing Solr
======================================================================================

Solr is a search engine that supports full-text search of indexed documents, plus features for categorization and data filtering. Managing Solr properly is critical to provide easy user access to uploaded Ontology objects. Solr should not be directly dealed with, but through REST services that read from JSON Mapping specifications (already introduced in Configuration).

Solr path must be properly set in SOLR_PATH Configuration variable.

Clear
------------------

Clears all indexed data (commits automatically)

::

    Service path: http://{host:port}/{appname}/solr/clear
    HTTP Method: GET
    Returns: "success" or "error"

Commit
----------------

Commits recent indexation

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

Solr indexation can be found at (SOLR_PATH)/data/data.xml

::

    Service path: http://{host:port}/{appname}/solr/reload
    HTTP Method: GET
    Returns: "success" or "error"

Update
----------------

Indexates new data according to mapping JSON configuration. This process may take some time.

Solr indexation can be found at (SOLR_PATH)/data/data.xml

::

    Service path: http://{host:port}/{appname}/solr/update
    HTTP Method: GET
    Returns: "success" or "error"

Search
----------------

Peforms search, JSON result comes straight from Solr engine.

::

    Service path: http://{host:port}/{appname}/solr/search?s=Search&f=Filter&start=Start&rows=Rows
    HTTP Method: GET
    Returns: Solr JSON result or "error"

"s" Search parameter, contains general text search.

"f" Filter parameter, defines results filtering in a "key:value" form sepparated by comas (see example below).

"start" and "rows" parameters define result scoping to provide pagination.

**Search Example**

::

    http://internetdomain.org/rest-path/solr/search?s=James

**OK Result**

::

    {
        + "responseHeader": { ... },                    // Not rellevant
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

Note that search results consists in identifiers only (they are always indexed as 'id'), to get the full object data we use View services

**How filtering works**

Constructing filter parameter is quite simple. All we have to do is to chose what category value we want to filter from "facet_fields" section of JSON Solr result. Still in previous example, say we want to filter by Birth year and Country with specific values for each. Filter could be: "Birth:1930,Country:United Kingdom".

**Filtering Example**

::

    http://internetdomain.org/rest-path/solr/search?s=James&f=Birth:1930,Country:United%20Kingdom   // Note that this may require URI encoding

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

