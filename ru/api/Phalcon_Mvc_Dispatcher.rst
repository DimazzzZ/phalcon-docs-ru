Class **Phalcon\\Mvc\\Dispatcher**
==================================

*extends* :doc:`Phalcon\\Dispatcher <Phalcon_Dispatcher>`

Dispatching is the process of taking the request object, extracting the module name, controller name, action name, and optional parameters contained in it, and then instantiating a controller and calling an action of that controller. 

.. code-block:: php

    <?php

    $di = new Phalcon\DI();
    
    $dispatcher = new Phalcon\Mvc\Dispatcher();
    
      $dispatcher->setDI($di);
    
    $dispatcher->setControllerName('posts');
    $dispatcher->setActionName('index');
    $dispatcher->setParams(array());
    
    $controller = $dispatcher->dispatch();



Methods
---------

public  **setControllerSuffix** (*string* $controllerSuffix)

Sets the default controller suffix



public  **setDefaultController** (*string* $controllerName)

Sets the default controller name



public  **setControllerName** (*string* $controllerName)

Sets the controller name to be dispatched



public *string*  **getControllerName** ()

Gets last dispatched controller name



protected  **_throwDispatchException** ()

Throws an internal exception



public :doc:`Phalcon\\Mvc\\Controller <Phalcon_Mvc_Controller>`  **getLastController** ()

Returns the lastest dispatched controller



public :doc:`Phalcon\\Mvc\\Controller <Phalcon_Mvc_Controller>`  **getActiveController** ()

Returns the active controller in the dispatcher



public  **__construct** () inherited from Phalcon\\Dispatcher

...


public  **setDI** (:doc:`Phalcon\\DI <Phalcon_DI>` $dependencyInjector) inherited from Phalcon\\Dispatcher

Sets the dependency injector



public :doc:`Phalcon\\DI <Phalcon_DI>`  **getDI** () inherited from Phalcon\\Dispatcher

Returns the internal dependency injector



public  **setEventsManager** (:doc:`Phalcon\\Events\\Manager <Phalcon_Events_Manager>` $eventsManager) inherited from Phalcon\\Dispatcher

Sets the events manager



public :doc:`Phalcon\\Events\\Manager <Phalcon_Events_Manager>`  **getEventsManager** () inherited from Phalcon\\Dispatcher

Returns the internal event manager



public  **setActionSuffix** (*string* $actionSuffix) inherited from Phalcon\\Dispatcher

Sets the default action suffix



public  **setDefaultNamespace** (*string* $namespace) inherited from Phalcon\\Dispatcher

Sets the default namespace



public  **setDefaultAction** (*string* $actionName) inherited from Phalcon\\Dispatcher

Sets the default action name



public  **setActionName** (*string* $actionName) inherited from Phalcon\\Dispatcher

Sets the action name to be dispatched



public *string*  **getActionName** () inherited from Phalcon\\Dispatcher

Gets last dispatched action name



public  **setParams** (*array* $params) inherited from Phalcon\\Dispatcher

Sets action params to be dispatched



public *array*  **getParams** () inherited from Phalcon\\Dispatcher

Gets action params



public  **setParam** (*mixed* $param, *mixed* $value) inherited from Phalcon\\Dispatcher

Set a param by its name or numeric index



public *mixed*  **getParam** (*mixed* $param) inherited from Phalcon\\Dispatcher

Gets a param by its name or numeric index



public *boolean*  **isFinished** () inherited from Phalcon\\Dispatcher

Checks if the dispatch loop is finished or have more pendent controllers/tasks to disptach



public *mixed*  **getReturnedValue** () inherited from Phalcon\\Dispatcher

Returns value returned by the lastest dispatched action



public *object*  **dispatch** () inherited from Phalcon\\Dispatcher

Dispatches a handle action taking into account the routing parameters



public  **forward** (*array* $forward) inherited from Phalcon\\Dispatcher

Forwards the execution flow to another controller/action



