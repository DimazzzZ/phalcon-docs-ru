Interface **Phalcon\\Annotations\\ReaderInterface**
===================================================

Phalcon\\Annotations\\ReaderInterface initializer


Methods
---------

abstract public *array*  **parse** (*string* $className)

Reads annotations from the class dockblocks, its methods and/or properties



abstract public static *array*  **parseDocBlock** (*string* $docBlock, *unknown* $file=null, *unknown* $line=null)

Parses a raw doc block returning the annotations found



