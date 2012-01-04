Ús del Solr
======================================================================================

Solr és un motor de cerca sobre els documents indexats, més característiques per categorització i filtrat de resultats. Utilitzar Solr adequadament és primordial per tal de proveir als usuaris d'un accés fàcil als objectes ontològics. Solr no hauria de ser utilitzat directament, sino a través del serveis REST que parteixen de les especificacions de mapeig JSON (ja esmentades a la Configuració).   

Clear
------------------

Neteja totes les dades indexades.

::

    Ruta servei: http://{host:port}/{appname}/solr/clear
    Mètode HTTP: GET
    Retorna: "success" o "error"

Commit
----------------

Realitza commit (innecessari ja que és automàtic)

::

    Ruta servei: http://{host:port}/{appname}/solr/commit
    Mètode HTTP: GET
    Retorna: media file

Schema
----------------

Genera el "schema" de la configuració de mapeig JSON.

::

    Ruta servei: http://{host:port}/{appname}/solr/clear
    Mètode HTTP: GET
    Retorna: "success" o "error"

NOTA: El solr ha de ser reiniciat per tal de carregar el nou esquema. 

Reload
----------------

Realitza una actualització (servei update del Solr) esborrant totes les dades prèvies. Aquest procés por trigar una mica.

La indexació Solr resultant es desa a l'arxiu: (SOLR_PATH)/data.xml

::

    Ruta servei: http://{host:port}/{appname}/solr/reload
    Mètode HTTP: GET
    Retorna: "success" o "error"

Update
----------------

Realitza una actualització (servei update del Solr) mantenint les dades prèvies. Aquest procés por trigar una mica.

La indexació Solr resultant es desa a l'arxiu: (SOLR_PATH)/data.xml

En principi aquesta crida no es recomana ja que poden romandre documents indexats d'objectes inexistents a l'ontologia.

::

    Ruta servei: http://{host:port}/{appname}/solr/update
    Mètode HTTP: GET
    Retorna: "success" or "error"

Search
----------------

Realitza la cerca, en aquest cas el resultat JSON ve directament del motor Solr.

::

    Ruta servei: http://{host:port}/{appname}/solr/search?s=Search&f=Filter&start=Start&rows=Rows
    Mètode HTTP: GET
    Retorna: Resultat JSON o "error"

"s" Search parameter, contains general text search.

"f" Filter parameter, defines results filtering in a "key:value" form sepparated by comas (see example below).

"start" and "rows" parameters define result scoping to provide pagination.

"sort" permet ordenar els resultats. El valor ha de ser el nom del camp d'ordenació més "desc" o "asc" segons el cas. Qulsevol camp usat aquí ha de tenir la clàusula "sort" a l'arxiu mapping.json.

**Exemple de cerca**

::

    http://internetdomain.org/rest-path/solr/search?s=James

**Resultat OK**

::

    {
        + "responseHeader": { ... },                    // Irrellevant
        - "response":
            {
                "numFound": 277,
                "start": 0,
              - "docs": [
                  - {
                        "id": "James_Bond_ID"           // Fixa't que els resultats contenen només l'identificador, per obtenir totes les dades de cada objecte utilitzem el servei "View"
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

             - "facet_fields": {                        // Amb el mapeig adequat, podem obtenir la categorització dels resultats (veure Configuració)
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

**Com filtrar els resultats**

El paràmetre de filtre "f" es pot construir de forma simple. Dels camps que s'inclouen dins el bloc "facet_fields" del resultat Solr, n'utilitzem el nom i un valor com a criteri de filtre. Partint de l'exemple anterior, suposem que volem filtrar per any de neixement i país amb valors especifics. Per exemple: "Birth:1930,Country:United Kingdom". 
Constructing filter parameter is quite simple. All we have to do is to chose what category value we want to filter from "facet_fields" section of JSON Solr result. Still in previous example, say we want to filter by Birth year and Country with specific values for each. Filter could be: "Birth:1930,Country:United Kingdom".

**Exemple de filtrat**

::

    http://internetdomain.org/rest-path/solr/search?s=James&f=Birth:1930,Country:United+Kingdom   // Els espais es poden substituir per signe "+"

**Resultat OK**

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

Si es filtra per dos valors de la mateixa categoria (per exemple: "Year:1930,Year:1975"), aquest filtre serà no exclusiu. 

Autocomplete
------------------------

Realitza un autocompletar donada una cadena de text. El JSON resultat ve directament del motor Solr.

::

    Ruta servei: http://{host:port}/{appname}/solr/autocomplete?s=Search
    Mètode HTTP: GET
    Retorna: Resultat JSOn o "error"

"s" Paràmetre de cerca que contindrà la cadena de text

**Example d'autocompletar**

::

    http://internetdomain.org/rest-path/solr/autocomplete?s=Ja

**Resultat OK**

::

    {
        + "responseHeader": { ... },                    // Irrellevant
        + "response": { ... },                          // En autocompletar la resposta principal també és irrellevant
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

Fixa't que en la cerca d'autocompletar, els resultats "facet" comprenen els camps marcats com "autocomplete" al mapping.json (veure Configuració).

Configuracions de cerca
-------------------------------

És possible personalitzar modes de cerca des del servidor. Això es fa editant o creant l'arxiu search.json al directori CONFIGURATIONS_PATH/mapping/.
Configurar cerques permet afegir filtrat addicional a les peticions del client per focalitzar els resultats a una vista particular.
Suposem que volem que les cerques focalitzin només a persones i països, i per neixements entre 1900 i 1950. L'adequada configuració del search.json quedaria així:

::
    
    {
        "name":"mycustomsearch"
        "type":"search",
        "value":["ObjectType:Person", "ObjectType:Country", "Birth:[1900 TO 1950]" ]
    },

    {
        "name":"default"
        "type":"search"
        // cerca per defecte, es pot deixar en blanc si no volem filtrat adicional
    }

Per usar aquesta configuració, utilitzem el paràmetre "config" amb valor "mycustomsearch"

**Example Cerca**

::

    http://internetdomain.org/rest-path/solr/search?s=James&config=mycustomsearch

Si la configuració no s'especifica, l'utilitza la configuració "default". Si no hi ha el fitxer search.json, no hi ha filtrat adicional.


