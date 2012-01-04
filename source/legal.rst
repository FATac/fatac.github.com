Legal
==================================

The purpose of this section is to guide user through the process of protecting objects from unauthorized access regarding their license and/or administrator preferences. First steps are explained in Configuration section and self-explained in legal.json script.

Legal accessing
------------------

In the legal process, the user will go through a scripted form (see Configuration) that will require the data to be resolved to a color result. This color will be used to determine who can access to object. You can assign access authorization to Plone groups by defining their Plone group names in Configuration variables USER_LEVEL.

**Example**

(Configuration properties)
::

    USER_LEVEL = *, MyUserGroup1, MyUserGroup2 MyUserGroup3, MyUserGroup4

In this example, all users (specified by "*" character) are given "green" access. Users of MyUserGroup1 are given "yellow" access. Users of MyUserGroup2 and MyUserGroup3 are given "orange" access. Finally, MyUserGroup4 users are given "red" access. Note that any user of more restrictive authorization color can also access to less restrictive authorization colors.

Services legally involved
-------------------------

These are the services that implement access restrictions according to object assigned color:

::

    /resource/{identifier}
    /resource/{identifier}/view
    /resource/{identifier}/thumbnail
    /media/{identifier}

They all do accept parameter "uid", its value must be current user identifier. This user identifier is internally used to resolve user group which is used to get user access level. If user level is equal or greater than object color level, this can be viewed/accessed. Objects with no access color can be viewed by anyone. If "uid" is not specified, access level is minimum ("green" and no color).

All data modification services which are:

::

    /resource/{identifier}/upload
    /resource/{identifier}/update
    /resource/{identifier}/delete
    /media/upload

Require that user group is the one with maximum priviledges, this is "red" authorization level.

Legal process
-----------------

Legal process is the script execution which results to a object color assignation. This process is run by REST services which are:

::

    Service path: http://{host:port}/{appname}/legal/start
    HTTP Method: POST
    Returns: JSON code of generated temporary user identifier and first form

POST body must contain JSON list of Object identifiers that will be processed

::

    Service path: http://{host:port}/{appname}/legal/next
    HTTP Method: POST
    Returns: JSON code of next form or result

POST body must contain JSON filled form and temporary user identifier.

::

    Service path: http://{host:port}/{appname}/legal/restore?key=KEY&value=VALUE
    HTTP Method: GET
    Returns: JSON code of generated temporary user identifier and first form

Support service that retrieves information concerning a specified key-value pair. This key must be specified at legal script using "autodata" clause. This should be called background (using AJAX) to fill current form with recurrent data to save user time from data introduction. 
