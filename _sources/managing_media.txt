Managing media
======================================================================================

AC REST services provide media storage and some format conversion cappabilities.

Upload
------------------

Uploads media file and proceeds to conversion if necessary (depending on Configuration)

::

    Service path: http://{host:port}/{appname}/media/upload?fn=FILE NAME
    HTTP Method: POST
    Accepted content: (any media type)
    Returns: full media URL or "error"

Http request body must contain binary file data.

Parameter "fn" is required, and it is the file name including format extension.

**POST Example**

::

    http://myhost:8080/rest/media/upload?fn=myimage.jpg

::

    [Media file binary content]

**OK Result**

::

    "http://myhost:8080/rest/media/myimage_276yt5o9l81cn8.jpg"       // URL with auto-generated id


Get
----------------

Retrieves stored media

::

    Service path: http://{host:port}/{appname}/media/{identifier}
    HTTP Method: GET
    Returns: media file
    
Identifier can contain profile index if media is eligible for conversion (such as audio and video), this is the file name with "_master", or "_1", "_2", etc.

**GET Example**

::

    http://myhost:8080/rest/media/myimage_276yt5o9l81cn8.jpg

**OK Result**

::

    [Media file binary content]
    
**GET Example (with profiles)**

::

    http://myhost:8080/rest/media/myvideo_chy9laj842qg3v_master.jpg
    http://myhost:8080/rest/media/myvideo_chy9laj842qg3v_1.jpg
    http://myhost:8080/rest/media/myvideo_chy9laj842qg3v_2.jpg

**OK Result**

::

    [Media file binary content]

Get format
---------------

Media file extension is also requestable

::

    Service path: http://{host:port}/{appname}/media/{identifier}/format
    HTTP Method: GET
    Returns: file extension

**GET Example**

::

    http://myhost:8080/rest/media/myimage_276yt5o9l81cn8.jpg/format

**OK Result**

::

    jpg

Convert
--------------

Proceed to media conversion (if applicable, this is having defined this media format in MEDIA_CONVERSION_PROFILES)

::

    Service path: http://{host:port}/{appname}/media/{identifier}/convert
    HTTP Method: GET
    Returns: "success" or "error"

Upon conversion, a "master" (this is, a copy of the original) media file will be created and a converted file will be generated for each applicable profile. To access each generated media we use a profile clause as described next.


Delete
---------------

Deletes stored media and its converted files

::

    Service path: http://{host:port}/{appname}/media/{identifier}/delete
    HTTP Method: DELETE
    Returns: "success" or "error"