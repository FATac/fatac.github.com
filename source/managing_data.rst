Managing data
======================================================================================

AC REST services provide CRUD access to data. All communication is in JSON format.

Note: data service base path should correspond to Configuration var RESOURCE_URI_NS plus the actual physical domain url.


Upload
------------------

Creates a new object of specific type and data

::

    Service path: http://{host:port}/{appname}/resource/upload
    HTTP Method: POST
    Accepted content: */json
    Returns: Identifier of uploaded object or "error"

Http request body must contain a JSON list of key-value tuples. 

"type" is a required key, otherwise returns error. 

"about" is a recommended key in order to generate proper object identifiers.

Multilingual properties can be specified with language code prefix for each value (see example).

**POST Example**

::

    http://internetdomain.org/rest-path/resource/upload

::

    {
        "type":"Location",
        "about":"New York City",
        "name":"New York City@en",
        "name":"Nueva York@es",
        "abbreviation":"NYC"
    }

**OK Result**

::

    "New_York_City"


Read
-------------------

Retrieves specific object data

::

    Service path: http://{host:port}/{appname}/resource/{identifier}
    HTTP Method: GET
    Returns: JSON object data

**GET Example**

::

    http://internetdomain.org/rest-path/resource/New_York_City

**OK Result**

::

    {
        "type":"Location",
        "about":"New York City",
        "name":"New York City@en",
        "name":"Nueva York@es",
        "abbreviation":"NYC"
    }


Update
-----------------------

Updates specified object data. 

::

    Service path: http://{host:port}/{appname}/resource/{identifier}/update
    HTTP Method: PUT
    Accepted content: */json
    Returns: "success" or "error"

Http request body must contain a JSON list of key-value tuples. 

In order to remove any existing property, this must be sent with empty value. Unsent properties remain unchanged.

Identifier and type cannot be changed.

**POST Example**

::

    http://internetdomain.org/rest-path/resource/New_York_City/update

::

    {
        "type":"Person",            // has no effect
        "name":"City of New York"   // changes property value (and turns to a single-language property)
        "abbrevitation":"",         // removes property
        "population":"8175133",     // creates property
    }

**OK Result**

::

    "success"


Delete
---------------------

Deletes specified object

::

    Service path: http://{host:port}/{appname}/resource/{identifier}/delete
    HTTP Method: DELETE
    Returns: "success" or "error"




