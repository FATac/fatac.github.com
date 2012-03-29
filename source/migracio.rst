Migració
====================================

El programa de migració es pot descarregar des de:

 https://github.com/FATac/ArtsCombinatoriesMigracio/blob/develop/migracio.tar.gz (cal descomprimir-lo)

Els paràmetres de la migració són els següents

./java -jar Migracio.jar <Què migrar> <Url-servidor> <Data-Hora servidor> <DirectoriMedias>

La migració es realitza amb els passos següents

**Pas 1**::

	./java -jar Migracio.jar "nomes dades" http://urlservidor.com:8080 "06/02/12 12:37" ""

Aquesta primera crida executa la migració de totes les dades. La url i la data i hora han de coincidir amb el servidor. El directori de medias no cal especificar-lo en aquest pas. Aquesta fase genera els arxius imagesExpedient.json, mediaFileId.json, objectExpedient.json, workExpedient.json i collectionList.json que són utilitzats pel programa de migració en el pas següent. 

**Pas 2**::

	./java -jar Migracio.jar "nomes media" http://urlservidor.com:8080 "" "/directori/medias/"
	
Ara no cal especificar la data i hora del servidor, però sí el directori dels medias. Si hi ha massa medias, es pot repetir el Pas 2 repetidament per cada grup de medias. Per assegurar-se que no puja dues vegades el mateix media, cal conservar l'arxiu "fitxerspujats.txt" on el programa va afegint tots els medias que va migrant. 

**Important**: A l'arxiu fitxerspujats.txt es guarda la ruta completa de cada arxiu pujat anteriorment, pertant si un mateix arxiu està ubicat en dos llocs diferents ho interpreta com que són arxius diferents. Cal tenir en compte això alhora de migrar medias des d'origens diferents (des d'un dispositiu, carpeta local, etc.). Podria ser necessari editar l'arxiu per substituir totes les rutes amb el nou origen.

Per exemple, si tenim una línia de fitxerspujats.txt com::
	
	/home/usuari/els_meus_medias/fotografia399.jpg
	
I intentem pujar /media/dispositiu/fotografia399.jpg, s'interpretarà com una nova fotografia perquè la ruta és diferent. Per tant cal modificar l'arxiu fitxerspujats.txt perquè quedi com::

	/media/dispositiu/fotografia399.jpg
	
(Amb un buscar-substituir es fa ràpid)


	


 

