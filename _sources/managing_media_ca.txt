Gestió de media
======================================================================================

Els serveis REST d'Arts Combinatòries permeten emmagatzematge i conversió de formats d'arxius media.

Upload
------------------

Carrega un fitxer media i procedeix a conversió si aplica (segons la configuració)

::

    Ruta servei: http://{host:port}/{appname}/media/upload?fn=NOM FITXER
    Mètode HTTP: POST
    Contingut acceptat: (qualsevol)
    Retorna: la URL completa del media o bé error

El cos de la petició HTTP ha de contenir la informació binària de l'arxiu.

El paràmetre "fn" és necessari, ha de ser el nom de l'arxiu incloent el format.

**Exemple POST**

::

    http://myhost:8080/rest/media/upload?fn=myimage.jpg

::

    [Contingut binari de l'arxiu media]

**Resultat OK**

::

    "_media_file_123"       // id auto-generat


Get
----------------

Retorna el fitxer media

::

    Ruta servei: http://{host:port}/{appname}/media/{identifier}
    Mètode HTTP: GET
    Retorna: arxiu media

L'identifier pot contenir l'index del perfil si el media és apte per a conversió (com l'àudio i el vídeo), això és un nom d'arxiu amb "_master, o "_1, "_2", etc.

**Exemple GET**

::

    http://myhost:8080/rest/media/myimage_276yt5o9l81cn8.jpg

**Resultat OK**

::

    [Contingut binari de l'arxiu media]
    
**Exemple GET (amb perfils)**

::

    http://myhost:8080/rest/media/myvideo_chy9laj842qg3v_master.jpg
    http://myhost:8080/rest/media/myvideo_chy9laj842qg3v_1.jpg
    http://myhost:8080/rest/media/myvideo_chy9laj842qg3v_2.jpg

Get format
---------------

També es pot sol·licitat el format de l'arxiu

::

    Ruta servei: http://{host:port}/{appname}/media/{identifier}/format
    Mètode HTTP: GET
    Retorna: extensió de l'arxiu

**Exemple GET**

::

    http://myhost:8080/rest/media/_media_file_123/format

**Resultat OK**

::

    jpg

Convert
--------------

Procedeix a la conversió del media (si aplica, d'acord amb allò especificat a la variable de configuració MEDIA_CONVERSION_PROFILES)

::

    Ruta servei: http://{host:port}/{appname}/media/{identifier}/convert
    Mètode HTTP: GET
    Retorna: "success" o "error"

Un cop convertit, es crearà un arxiu "master" (és a dir, una còpia de l'original), i es crearà un arxiu convertit per a cada perfil aplicable. Per accedir a cada media utilitzem el següent servei.



Delete
---------------

Esborra el media i els seus arxius convertits

::

    Ruta servei: http://{host:port}/{appname}/media/{identifier}/delete
    Mètode HTTP: DELETE
    Retorna: "success" o "error"