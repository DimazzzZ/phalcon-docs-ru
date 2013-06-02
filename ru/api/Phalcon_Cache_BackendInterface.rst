Interface **Phalcon\\Cache\\BackendInterface**
==============================================

Phalcon\\Cache\\BackendInterface initializer


Methods
---------

abstract public *mixed*  **start** (*int|string* $keyName, *long* $lifetime=null)

Starts a cache. The $keyname allows to identify the created fragment



abstract public  **stop** (*boolean* $stopBuffer=null)

Stops the frontend without store any cached content



abstract public *mixed*  **getFrontend** ()

Returns front-end instance adapter related to the back-end



abstract public *array*  **getOptions** ()

Returns the backend options



abstract public *boolean*  **isFresh** ()

Checks whether the last cache is fresh or cached



abstract public *boolean*  **isStarted** ()

Checks whether the cache has starting buffering or not



abstract public  **setLastKey** (*string* $lastKey)

Sets the last key used in the cache



abstract public *string*  **getLastKey** ()

Gets the last key stored by the cache



abstract public *mixed*  **get** (*int|string* $keyName, *long* $lifetime=null)

Returns a cached content



abstract public  **save** (*int|string* $keyName=null, *string* $content=null, *long* $lifetime=null, *boolean* $stopBuffer=null)

Stores cached content into the file backend and stops the frontend



abstract public *boolean*  **delete** (*int|string* $keyName)

Deletes a value from the cache by its key



abstract public *array*  **queryKeys** (*string* $prefix=null)

Query the existing cached keys



abstract public *boolean*  **exists** (*string* $keyName=null, *long* $lifetime=null)

Checks if cache exists and it hasn't expired



