Class **Phalcon\\Mvc\\View\\Engine\\Php**
=========================================

*extends* :doc:`Phalcon\\Mvc\\View\\Engine <Phalcon_Mvc_View_Engine>`

Adapter to use PHP itself as templating engine


Methods
---------

public  **render** (*string* $path, *array* $params, *bool* $mustClean)

Renders a view using the template engine



public  **__construct** (:doc:`Phalcon\\Mvc\\View <Phalcon_Mvc_View>` $view, :doc:`Phalcon\\DI <Phalcon_DI>` $dependencyInjector) inherited from Phalcon\\Mvc\\View\\Engine

Phalcon\\Mvc\\View\\Engine constructor



public *array*  **getContent** () inherited from Phalcon\\Mvc\\View\\Engine

Returns cached ouput on another view stage



public *string*  **partial** (*string* $partialPath) inherited from Phalcon\\Mvc\\View\\Engine

Renders a partial inside another view



public  **setDI** (:doc:`Phalcon\\DI <Phalcon_DI>` $dependencyInjector) inherited from Phalcon\\DI\\Injectable

Sets the dependency injector



public :doc:`Phalcon\\DI <Phalcon_DI>`  **getDI** () inherited from Phalcon\\DI\\Injectable

Returns the internal dependency injector



public  **setEventsManager** (:doc:`Phalcon\\Events\\Manager <Phalcon_Events_Manager>` $eventsManager) inherited from Phalcon\\DI\\Injectable

Sets the event manager



public :doc:`Phalcon\\Events\\Manager <Phalcon_Events_Manager>`  **getEventsManager** () inherited from Phalcon\\DI\\Injectable

Returns the internal event manager



public  **__get** (*string* $propertyName) inherited from Phalcon\\DI\\Injectable

Magic method __get



