Gestió de dades
======================================================================================

Els serveis REST d'Arts Combinatòries proporcionen accés CRUD a les dades. Tota comunicació es fa en format JSON.

Upload
------------------

Crea un nou objecte d'un tipus i amb un conjunt de dades determinades

::

    Ruta servei: http://{host:port}/{appname}/resource/upload
    Mètode HTTP: PUT
    Contingut acceptat: */json
    Retorna: L'identificador de l'objecte creat o bé "error"

El cos de la crida HTTP ha de contenir una llista JSON de tuples nom-valor. 

"type" o "rdf:type" és una dada requerida, altrament retorna error. 

"about" és una dada recomanada per tal de generar adequadament l'identificador.

Les propietats poden contenir valors multilingües, en aquest cas cal incloure el prefix d'idioma (veure exemple).

**Exemple PUT**

::

    http://internetdomain.org/rest-path/resource/upload

::

    {
        "type":"Location",
        "about":"New York City",
        "my:name":"New York City@en",
        "my:name":"Nova York@ca",
        "my:abbreviation":"NYC"
    }
    
El prefix d'idioma és '@' més el codi d'idioma.

**Resultat OK**

::

    "New_York_City"


Read
-------------------

Retorna les dades d'un objecte específic

::

    Ruta servei: http://{host:port}/{appname}/resource/{identifier}
    Mètode HTTP: GET
    Retorna: dades JSON

**Exemple GET**

::

    http://internetdomain.org/rest-path/resource/New_York_City

**Resultat OK**

::

    {
        "rdf:type":"Location",
        "about":"New York City",
        "my:name":"New York City@en",
        "my:name":"Nueva York@es",
        "my:abbreviation":"NYC"
    }


Update
-----------------------

Actualitza les dades d'un objecte específic.

::

    Ruta servei: http://{host:port}/{appname}/resource/{identifier}/update
    Mètode HTTP: POST
    Contingut acceptat: */json
    Retorna: "success" o "error"

El cos de la crida HTTP ha de contenir una llista JSON de tuples nom-valor.

Per tal d'esborrar alguna propietat existent, aquesta s'ha d'enviar amb un valor buit. Les propietats no enviades no pateixen cap modificació.

L'identificador i el tipus no es poden canviar.

**Exemple POST**

::

    http://internetdomain.org/rest-path/resource/New_York_City/update

::

	// Partint de l'exemple anterior
    {
        "type":"Person",            // no té cap efecte (el mateix per "rdf:type")
        "name":"City of New York"   // canvia el valor (i passa a tenir un sol valor)
        "abbrevitation":"",         // esborra la propietat
        "population":"8175133",     // crea la propietat
    }

**Resultat OK**

::

    "success"


Delete
---------------------

Esborra un objecte específic

::

    Ruta servei: http://{host:port}/{appname}/resource/{identifier}/delete
    Mètode HTTP: DELETE
    Retorna: "success" o "error"




