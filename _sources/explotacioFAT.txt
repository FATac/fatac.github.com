.. FAT Arts Combinatòries documentation master file, created by
   sphinx-quickstart on Tue May 31 12:39:26 2011.
   You can adapt this file completely to your liking, but it should at least
   contain the root `toctree` directive.

Notes d'explotació d'Arts Combinatòries a la Fundació Antoni Tàpies
======================================================================================

Protocol de modificació de patrons, mapeig o flux legal
-------------------------------------------------------------------

Els arxius de configuració s'allotgen a "/var/plone/artscombinatories/config/" del servidor que correspon a un repositori Github "ArtsCombinatoriesConfiguracio". Per modificar qualsevol arxiu i fer efectius els canvis caldrà seguir els passos següents:

Si és el primer cop que ens disposem a modificar la configuració, cal obtenir el projecte amb els següents passos:

 #. (Anar al nostre directori)
 #. (Crear la carpeta "ArtsCombinatoriesConfiguracio")
 #. cd ArtsCombinatoriesConfiguracio
 #. git init
 #. git remote add origin git@github.com:FATac/ArtsCombinatoriesConfiguracio.git
 #. git pull --all

Quan ja tenim el projecte baixat, i després de fer les pertinents modificacions als arxiu JSON:

 #. (Si hem afegit nous arxius JSON cal cridar) git add .
 #. git commit -a   (i desar el missatge de commit)
 #. git push

(Administrador UPCNET) Des del servidor, executar:

 #. cd /var/plone/artscombinatories/config/
 #. git pull
 #. (Si l'arxiu modificat és "mapping.json" cal reiniciar l'aplicació web "ArtsCombinatoriesRest" del Tomcat)