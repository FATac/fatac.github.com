Legal
==================================

El propòsit d'aquesta funcionalitat és guiar l'usuari pel procés de protecció d'objectes d'un accés no autoritzat, d'acord amb la seva llicència o preferències d'administració. Els primers passos s'expliquen a la Configuració i a l'arxiu legal.json.

Accés legal
------------------

En el procés legal, l'usuari haurà de passar per un flux configurat (veure Configuració) que requerirà que les dades condueixin a un color resultant. Aquest color serà utilitzat per a determinar qui pot accedir a l'objecte. Pots assignar autoritzacions d'accés als grups de Plone definint els noms del grup a la variable de Configuració USER_LEVEL.

**Exemple**

(config.json)
::

    "USER_LEVEL": { "*", "MyUserGroup1", "MyUserGroup2+MyUserGroup3", "MyUserGroup4" }

En aquest exemple, tots els usuaris (especificats amb '*') tenen accés verd. Els usuaris de MyUserGroup1 accés groc. Els usuaris de MyUserGroup2 i MyUserGroup3 accés taronja. I els usuaris de MyUserGroup4 accés vermell. S'entén que qualsevol usuari també pot accedir als objectes de nivells d'autorització (colors) inferiors al del seu grup.

Serveis que implementen les restriccions legals
----------------------------------------------------

La finalitat de les funcions legals són en definitiva protegir els continguts audiovisuals, per aquest motiu els serveis implicats són aquells que retornen aquest tipus de contingut o la seva URL:

::

	/resource/{identifier}/thumbnail
	/resource/{identifier}/view
	/resource/{identifier}
	/media/{identifier}

Tots ells accepten el paràmetre "uid". El seu valor ha de ser l'identificador d'usuari, que servirà per identificar el seu grup. Si el nivell d'autorització del grup és igual o superior al de l'objecte al qual es vol accedir es permetrà l'accés als media. Els objectes que no tenen cap color assignat s'entén que són públics, equivalent a "verd". Si no s'especifica "uid", el nivell d'accés per defecte és "verd".

Tots els serveis de modificació de dades:

::

    /resource/{identifier}/upload
    /resource/{identifier}/update
    /resource/{identifier}/delete
    /media/upload

Requereixen que el grup de l'usuari tingui autorització màxima, és a dir "vermell".

Procés legal
-----------------

El procés legal és l'execució del flux que resulta en una assignació de color a l'objecte o objectes. Aquest procés s'executa fent crides als serveis REST, que són:

::

    Ruta servei: http://{host:port}/{appname}/legal/start
    Mètode HTTP: POST
    Retorna: Codi JSON amb un identificador d'usuari temporal

El cos de la crida ha de contenir una llista JSON dels identificadors dels objectes que seran processats.

::

    Ruta servei: http://{host:port}/{appname}/legal/next
    Mètode HTTP: POST
    Retorna: Codi JSON amb les dades del següent formulari o bé el resultat.

El cos de la crida ha de contenir els valors dels camps del formulari anterior juntament amb l'identificador temporal d'usuari.

::

    Ruta servei: http://{host:port}/{appname}/legal/restore?key=KEY&value=VALUE
    Mètode HTTP: GET
    Retorna: Codi JSON amb les dades associades a la tupla key-value

Servei de suport que retorna dades relacionadades amb la tupla key-value. El paràmetre key ha de ser especificat al flux legal (legal.json) usant la clàusula "reference". 
Aquest servei hauria de cridar-se per darrera (amb AJAX) per emplenar el formulari actual amb les dades corresponents i estalviar temps a l'usuari. 
