.. FAT Arts Combinatòries documentation master file, created by
   sphinx-quickstart on Tue May 31 12:39:26 2011.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

Configuració
======================================================================================

Cal definir diversos aspectes per tal que Arts Combinatòries (AC) funcioni amb una nova Ontologia. Aquesta pàgina mostra pas a pas el procediment de configuració.

Pas 1r: Virtuoso
---------------------------

És necessari establir la configuració incial del Virtuoso. Accedeix a la consola de Virtuoso Conductor (per defecte: http://localhost:8890/conductor/ amb usuari i contrassenya "dba"). Clicar a "Interactive SQL" a la dreta, i executar l'script SQL següent:

Some Virtuoso initial configuration is necessary. Access to Virtuoso Conductor console (default should be: http://localhost:8890/conductor/ with both user and password "dba"). Click "Interactive SQL" on the right, and run the next sql script:

::

    -- Base script which creates necessary data structure for managing users, rights and uploaded media and objects
	-- Suitable (only?) for Openlink Virtuoso sql implementation
	
	DROP TABLE db.dba._media;
	DROP TABLE db.dba._thumbnail;
	DROP TABLE db.dba._right;
	DROP TABLE db.dba._identifier_counter;
	DROP TABLE db.dba._autodata;
	DROP TABLE db.dba._resource_statistics;
	
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
	
	--GRANT EXECUTE ON DB.DBA.SPARQL_DELETE_DICT_CONTENT TO "SPARQL";
	GRANT ALL PRIVILEGES TO "SPARQL";

(Es pot descomentar els "drops" per netejar les taules)

A continuació, clic a la pestanya "RDF" i després "Namespaces". Cal afegir aquí totes les ontologies relacionades que no apareguin a la llista, aquestes ontologies són les especificades a la propietat "ONTOLOGY_NAMESPACES" (Pas 3r).

Pas 2n: Solr
---------------------------

Un cop instal·lat el Solr, cal utilitzar un schema.xml específic. Aquest es genera d'acord amb les especificacions de mapeig (veure Pas 6è) i d'acord amb una "schema" base. Aquest esquema base és un arxiu situat al directori "solr" (referenciat a la variable "SOLR_PATH" del Pas 3r), amb exactament el contingut següent:

::

	<?xml version="1.0" encoding="UTF-8" ?>
	
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
		      </analyzer>
		      <analyzer type="query">
		        <tokenizer class="solr.KeywordTokenizerFactory"/>
		        <charFilter class="solr.MappingCharFilterFactory" mapping="mapping-ISOLatin1Accent.txt"/>
		      </analyzer>
		    </fieldType>
		
		    <fieldType name="text_general" class="solr.TextField" positionIncrementGap="100">
		      <analyzer type="index">
		        <tokenizer class="solr.WhitespaceTokenizerFactory"/>
		        <filter class="solr.StopFilterFactory" words="stopwords.txt" ignoreCase="true"/>
		        <filter class="solr.LowerCaseFilterFactory" />
		        <charFilter class="solr.MappingCharFilterFactory" mapping="mapping-ISOLatin1Accent.txt"/>
		      </analyzer>
		      <analyzer type="query">
		        <tokenizer class="solr.WhitespaceTokenizerFactory"/>
		        <filter class="solr.StopFilterFactory" words="stopwords.txt" ignoreCase="true"/>
		        <filter class="solr.LowerCaseFilterFactory" />
		        <charFilter class="solr.MappingCharFilterFactory" mapping="mapping-ISOLatin1Accent.txt"/>
		      </analyzer>
		    </fieldType>
		 </types>
		
		<!-- FIELDS_INSERTION_MARK -->
		
		 <uniqueKey>id</uniqueKey>
		
		 <defaultSearchField>id</defaultSearchField>
		
		 <solrQueryParser defaultOperator="OR"/>
	
	</schema>

Pas 3er: Propietats principals
--------------------------------------

El primer que hem de fer per a configurar AC és definir l'arxiu de propietats "config.json". Aquest arxiu és el primer que s'utilitza i ha d'estar al directori actual (current directory). Si no sabeu quin és el directori actual podeu mirar el log d'AC en el moment d'inici de l'aplicació. El següent bloc mostra un exemple de les propietats requerides i possibles valors.

::

    {	
        "__comment_0":"Configuracio diversa",

	    "THUMBNAIL_WIDTH":250,
	    "THUMBNAIL_HEIGHT":180,
	    "MEDIA_CONVERSION_PROFILES":["dv", "mpg", "avi", "aif", "mov"],
        "MEDIA_AUTOCONVERT":"false",
	    "LANGUAGE_LIST":["ca", "en", "es", "fr", "it", "de"],							
	    "USER_LEVEL":["*", "Member", "Manager+Reviewer", "Site Administrator"],	    
	
	    "__comment_1":"Les URL base dels serveis",

	    "RDFDB_URL":"jdbc:virtuoso://myhost:1111",
	    "RDFDB_USER":"dba",
	    "RDFDB_PASS":"dba",
	    "REST_URL":"http://myhost:8080/rest/",
	    "SOLR_URL":"http://myhost:8080/solr/",
	    "VIDEO_SERVICES_URL":"http://myhost:8080/videoservices/rest/",
	    "USER_ROLE_SERVICE_URL":"http://myotherhost:8080/myapp/getUserRole?userId=",
	
        "__comment_2":"Les ontologies i els namespaces (qualsevo canvi implicarà corregir tots els registres existents a la BD)",

	    "RESOURCE_URI_NS":"http://localhost:8080/ArtsCombinatoriesRest/resource/",		
	    "RESOURCE_PREFIX":"ac_res",
	    "ONTOLOGY_NAMESPACES":[
		    "http://localhost:8080/rest/ontology/my#", "my",
		    "http://www.w3.org/1999/02/22-rdf-syntax-ns#", "rdf",
		    "http://www.w3.org/2000/01/rdf-schema#", "rdfs",
		    "http://dublincore.org/2010/10/11/dcterms.rdf#", "dcterms"
	    ],
	
	    "__comment_3":"Els directoris base on AC allotjarà o accedirà a continguts i configuracions",

	    "CONFIGURATIONS_PATH":"/achome/json/",
	    "SOLR_PATH":"/achome/solr/",
	    "MEDIA_PATH":"/achome/media/",
	    "ONTOLOGY_PATH":"/achome/myontology.owl"
    }

THUMBNAIL_WIDTH i THUMBNAIL_HEIGHT determina la mida de les miniatures generades.

MEDIA_CONVERSION_PROFILES enumera extensions vídeo/àudio adequats per una conversió, ordenats per número de perfil (p.ex: "dv" és perfil 1, "mpg" és perfil 2, etc.).

MEDIA_AUTOCONVERT posat a "true" si cal que cada cop que es pugi un fitxer mèdia, aquest sigui convertit automàticament d'acord amb els perfils i la propietat anterior. En altre cas sempre hi haurà disponible el servei "convert" (veure secció Gestió de Medias).

LANGUAGE_LIST enumera els codis d'idioma que s'espera que siguin emprats en els camps de les propietats (el primer de la llista serà el considerat com l'idioma d'accés per defecte).

USER_LEVEL especifica el grau d'accés legal que té cada rol d'usuari, ordenats de més a menys restricció ("*" significa qualsevol rol). Com que només hi ha 4 nivells de restricció, aquesta llista hauria de contenir sempre 4 elements. Cada element pot contenir més d'un rol, separat per '+' (p.ex: "Manager+Reviewer").

USER_ROLE_SERVICE_URL és una URL a un servei específic. Aquest servei és emprat per AC per obtenir els grups d'usuari que determinaran el permís d'accés de l'usuari. El servei ha d'acceptar un identificador (a la cadena de la URL) i hauria de retornar un dels grups d'usuari especificats a USER_LEVEL.

ONTOLOGY_NAMESPACES estableix un prefix per a cada namespace d'Ontologia, aquesta relació també ha d'aparèixer la llista "namespaces" del Virtuoso (veure Pas 1r). La primera ontologia especificada ha de ser necessariament aquella que hagi estat creada (o escollida) especialment per la web semàntica (corresponent a l'arxiu especificat a la variable "ONTOLOGY_PATH") i la resta d'ontologies seran aquelles importades a l'ontologia principal. Generalment, els esquemes RDF i RDFS haurien de ser inclosos sempre. 

AC requereix la següent estructura de directoris:

- [CONFIGURATIONS_PATH]
    - legal/legal.json (requerit)
    - mapping/mapping.json (requerit)
    - mapping/search.json (opcional)
    - mapping/ (opcionalment, un arxiu json per a cada classe de l'Ontologia, amb el seu prefix, per exemple "foaf:Person.json")
- [SOLR_PATH] (directori Home del Solr)
    - conf/schema.xml-EMPTY (requerit)
    - data/data.xml (autogenerat per l'aplicació cada cop que es faci una indexació a Solr, i contindrà les dades indexades)
- [MEDIA_PATH]
    - thumbnail/
    - thumbnail/classes/default.jpg (requerit. Miniatura per defecte de tots els objectes. No té perquè ser d'una mida específica)
    - thumbnail/classes/ (opcionalment, una miniatura per defecte per a cada classe amb el seu prefix, exemple "foaf:Person.jpg")
- [ONTOLOGY_PATH] (ruta completa a l'arxiu que conté l'ontologia principal del projecte)

Pas 4t: Reiniciar
-----------------------------

Cridant el servei "reset", TOTES les dades i arxius media seran esborrats. També s'actualitzarà l'Ontologia principal amb la darrera versió (situada a ONTOLOGY_PATH).

::

    Service path: http://{host:port}/{appname}/reset?option=ontology&confirm=CURRENT_DATE
    HTTP Method: GET
    Returns: "success" or "error"

Poseu "option=ontology" si no voleu un reinici total, sinó només una recàrrega de totes les ontologies especificades a ONTOLOGY_NAMESPACES.

En altre cas, per seguretat, cal omplir el paràmetre "confirm" amb la data i hora actual del servidor formatat com: "dd/mm/yy hh:mm"

**Exemples**

::

    http://internetdomain.org/ac/reset?option=ontology               // recarrega ontologies

::

    http://internetdomain.org/ac/reset?confirm=11/11/2011 23:11      // reinicia dades i recarrega ontologies



Pas 5è: Script legal
-----------------------------

AC proporciona funcionalitats per assignar drets legals als objectes media. L'assignació de drets és un procés assistit que pot ser guionat i completament personalitzat. (Si no tens intenció d'utilitzar aquesta característica pots obviar aquest pas)

Hi ha un exemple auto-explicatiu a "legal.json" al directori de configuració, subdirectori "legal". "legal.json" és el nom de l'arxiu que conté el guió que assistirà l'usuari. Les parts d'aquest guió són:

- StartBlock: bloc inicial
- Llista de Blocks: pels quals passarà l'execució del guió.
- Nom del block: serà usat per a referenciar-lo d'altres blocks.
- Descrició del block: finalitat simplement informativa.
- Dades del block: dades que seran sol·licitated a l'usuari (en forma de formulari) i seran utilitzades per a determinar l'assignació de drets. Aquestes dades són considerades globals, és a dir que poden ser usades o reassignades en blocks posteriors.
- Regles del block: evaluació lògica mitjançant expressions booleanes de les dades introduïdes en el procés. Aquesta evaluació por portar a un nou block indicat per la paraula "block" keyword, o a un color legal indicat per la paraula 'result'. Les conseqüències funcionals de cada color s'expliquen a continuació:

Hi ha quatre "colors de semàfor" que poden ser assignats a cada objecte. De menys a més restrictiu: "green", "yellow", "orange" i "red". Cada un correspon a un nivell d'accés de 1 a 4. Per cada crida a un servei que proveeix audiovisuals o urls a audiovisual,l'identificador d'usuari ha de ser especificiat, en funció d'això i de la configuració (variable USER_LEVEL) se n'obtindrà el nivell. Exemple. User level = 1 , Object level = 2 --> Fail / User level = 2 , Object level = 2 --> OK.

Pas 6è: Mapeig de dades
------------------------------

El fitxer "mapping.json" (situat a (CONFIGURATIONS_PATH)/mapping) és un arxiu necessari amb la definició de la forma com les dades seran indexades al Solr. El mapeig de dades no és una traducció plana dels camps i classes de l'ontologia. Cal especificar-lo de tal manera que després pugui ser usat pels diversos aspectes funcionals de la plataforma (patrons, ordenació, filtres, etc. que s'expliquen en aquest manual). 

Suposem que tenim la classe Person definida a la nostra Ontologia, i que volem indexar diverses dades com: nom, biografia, data de neixement, lloc de neixement. La indexació de Person serà especificada de la següent manera:

::

    {
	    "data":
	    [
            {
                "name":"Nom",                       // Especifica l'identificador de la dada, en aquest cas el Nom
                "type":"string",                    // 'string' indexa la dada com a un "string" (fieldType) del schema.xml del Solr 
                "path":["my:Person.my:fullName"]    // Ruta a propietat de la classe, especificat de la forma (prefix:Nom-classe).(prefix:propietat)
            },

            {
                "name":"Biografia",             
                "type":"text",                  	// 'text' indexa la dada com a "text_general" (fildType) del schema.xml del Solr.
                "path":["my:Person.my:Bio"]           
            },

            {
                "name":"DataNeixement",             
                "type":"date.year",             	// "date.year" extraurà l'any de la data continguda a la propietat (el format esperat és "dd/mm/yyyy" o simplement "yyyy")
                "path":["my:Person.my:BirthDate"]           
            },

            {
                "name":"LlocNeixement",             
                "type":"string",                
                "path":["my:Person.my:BirthPlace=my:Location.my:Name"]   // El lloc de neixement és de fet un objecte referenciat (Location), per aquest motiu cal encadenar les relacions dels objectes mitjançant '='. Per aquest mètode es poden encadenar tants objectes com calguin.
            }
        ]
    }

La ruta (path) és una llista, això permet especificar diverses rutes per un mateix camp indexat. Suposem que volem indexar noms de persones i sota una mateixa variable. El codi quedarà de la següent manera:

::

    "data":
        [
            {
                "name":"Nom",                                  
                "type":"string",                                
                "path":["my:Person.my:fullName", "my:Location.my:Name"]     // Ruta a propietats tant de Person com de Location
            },

            /* resta del json ... */
        ]

Per proporcionar les cerques adequades, podem establir clàusules adicionals per a cada dada:

- **category**: El Solr utilitzarà la característica 'facets' per categoritzar les dades implicades agrupant-les i comptant les coincidències.
- **sortCategory**: Si heu establert la clàusula anterior, els elements de la categoria s'ordenaran per número de coincidències. Es pot escollir l'ordenació alfabètica establint aquesta clàusula (és a dir, s'utilitzarà el paràmetre "facet.sort=index" del Solr).  
- **multilingual**: Aplicable a les propietats introduïdes en diversos idiomes a la base de dades RDF. Per exemple, la biografia d'una persona pot ser escrita en diversos idiomes. Això assegura que el Solr tornarà les dades només en l'idioma desitjat (veure secció Gestionant el Solr).
- **search**: Pot semblar obvi que totes les dades indexades han de ser utilitzades per la cerca, però no té perquè ser així (algunes son introduïdes només a efectes d'ordenació, i d'altres nomñes a efectes de categorització). Cal establir aquesta clàusula explícitament perquè la dada sigui utilitzada en la cerca.
- **autocomplete**: Si i només si heu establert la clàusula anterior, es pot optar per utilitzar la dada per autocompletar una cerca.
- **sort**: Si volem que un camp pugui ser utilitzat posteriorment per a ordenar els resultats de cerca, cal especificar-ho explícitament amb aquesta clàusula. Això causarà forçosament que aquest camp serà de valor únic (a diferència de la resta que són multi-valor) per a permetre l'ordenació, per aquest motiu generalment els camps destinats a ordenació no s'utilitzaran per a res més. 

Per exemple: la dada "Nom" abans descrita (nom de persona o lloc), és interessant tant per a cercar com per a autocompletar. Però el nom de Person és especificat en un sol idioma, i el nom de Location és especificat en diferents idiomes. A més, categoritzarem els resultats per llocs però no per persones. D'acord amb això, el codi json anterior canvia a:

::

    "data":
        [
            {
                "name":"Persona",                                  
                "type":"string",                                
                "path":["my:Person.my:fullName"],         
                "search":"yes",
                "autocomplete":"yes"
            },

            {
                "name":"Lloc",                                  
                "type":"string",                                
                "path":["my:Location.my:Name", "my:Person.my:BirthPlace=my:Location.my:Name"]
                "search":"yes",                     // Totes les clàusules són desactivades per defecte 
                "autocomplete":"yes",               // pel que han de ser especificades sempre que es necessitin
                "multilingual":"yes",
                "category":"yes"
            }

            /* resta del json ... */
        ]


Pas 7è: Patrons de classe
------------------------------------

Qualsevol cerca retornarà una llista d'IDs dels objectes que s'adeqüen als criteris de cerca. Per a poder generar la informació d'aquests objectes cal accedir individualment a cadascún mitjançant el servei "view" (veure secció Visualització), i per obtenir aquesta vista cal haver definit un patró per la classe d'aquest objecte. Per aquest motiu serà necessari que totes les classes referenciades d'arrel al "mapping.json" tinguin definit el seu patró corresponent. 

Tornant a l'exemple de la classe Person: el nom, data de neixement, i lloc de neixement els podem posar a la capçalera (header). La biografia al cos (body). I al peu (footer) hi podem incloure la classe 'knows', que relaciona persones entre si.

El template resultant ha d'anomenar-se "my:Person.json" (generalitzant, (prefix:Nom-classe).json) al directory mapping de CONFIGURATIONS_PATH. El codi quedaria de la següent manera:

::

    {
	    "className":"Person",
	
	    "sections":
	    [
		    {
			    "name":"header",                    // nom de secció
			    "data":[
		
		        	{
					    "name":"Nom",
					    "type":"text",
					    "path":["my:Person.my:fullName"]
				    },

                    {
					    "name":"DataNeixement",
					    "type":"date",
					    "path":["my:Person.my:BirthDate"]
				    },

                    {
					    "name":"LlocNeixement",
					    "type":"linkedObject",
					    "path":["my:Person.my:BirthPlace=my:Location.my:Name"]
				    }
			    ]
		    },
		
		    {
			    "name":"body",
			    "data":[
			
			     	{
			            "name":"Biografia",
			            "type":"text",
			            "path":["my:Person.my:Bio"]
			        }
			    ]  
		    },

            {
			    "name":"footer",                    
			    "data":[
		
		        	{
					    "name":"Relacionats",
					    "type":"search",
                        "path":["my:Person.id"],
                        "value":["RelatedPeople:"]
				    }
			    ]
		    }
	    ]
    }


El tipus de dada dels patrons és diferent del tipus explicat al pas anterior. Els tipus següents són els disponibles per a patrons:

- **text**: adequat per la majoria de casos, resol la ruta (path) a un literal sense cap modificació adicional.
- **linkedObject**: resol la ruta a un literal i hi afegeix l'id de l'objecte que el conté, separat per '@'. Això permet crear enllaços html entre objectes. Per exemple: Londres@london_id, l'enllaç referenciaria a http://myhost:8080/rest/resource/london_id/...
- **objects**: resol la ruta especificada on la propietat és de fet una relació, i pertant el resultat serà un identificador. 
- **media**: resol la ruta a una URL que conté un media.
- **date**: i les seves específiques (**date.year**, **date.day**, **date.month**). Anàlog al "date" explicat al pas 6è.
- **search**: aquest tipus combina la resolució de la ruta amb les cerques Solr per a generar conjunts de resultats per a l'usuari sobre els quals es puguin fer cerques adicionals. En aquest exemple: es cridarà una cerca que retornarà ("Person.knows:") that know current person ("Person.id"). Per més informació sobre aquest element si us plau llegeix la secció de Visualització.
- **counter**: resol la ruta i agrupa i fa un recompte de les coincidències.

Adoneu-vos que **text**, **objects** and **media** fan el mateix a la pràctica. La diferència és que el valor que resolen es suposa que és per propòsits diferents. Veure la secció Visualització per a més informació sobre els tipus **media** i **objects**.

Manteniment
----------------------------

Servei que realitza les següents tasques de manteniment regular de l'aplicació:

- Esborrar els fitxers temporals que es creen durant els processo legals
- Indexació de les dades a Solr (és a dir crida el servei /solr/reload)
- Actualització de les dades per a OAI-PMH (només si la variable OAI_PATH estigui definida)

::

    Ruta servei: http://{host:port}/{appname}/maintenance
    Mètode HTTP: GET
    Retorna: "success" o "error"
    
La crida d'aquest servei en un servidor a producció hauria de ser periòdica, a l'hora de menys tràfic d'usuaris/peticions.