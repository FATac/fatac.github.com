Managing media
======================================================================================

AC REST services provide media storage and some format conversion cappabilities.

Upload
------------------

Uploads media file and proceeds to conversion if necessary (eligible file formats for conversion are in Configuration var VIDEO_FILE_EXTENSIONS)

::

    Service path: http://{host:port}/{appname}/media/upload?fn=FILE NAME
    HTTP Method: POST
    Accepted content: (any media type)
    Returns: media identifier or "error"

Http request body must contain binary file data.

Parameter "fn" is required, and it is the file name including format extension.

**POST Example**

::

    http://internetdomain.org/rest-path/media/upload?fn=myimage.jpg

::

    [Media file binary content]

**OK Result**

::

    "_media_file_123"       // auto-generated id


Get
----------------

Retrieves stored media

::

    Service path: http://{host:port}/{appname}/media/{identifier}
    HTTP Method: GET
    Returns: media file

**GET Example**

::

    http://internetdomain.org/rest-path/media/_media_file_123

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

    http://internetdomain.org/rest-path/media/_media_file_123/format

**OK Result**

::

    jpg




