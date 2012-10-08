Using Dependency Injection
==========================
The following example is a bit lengthy, but explains why using a service container and dependency injection. To begin with, let's pretend we are developing a component called SomeComponent. This performs a task that is not important right now. Our component have some dependency that is a connection to a database.

In this first example, the connection is created inside the component. This approach is impractical, practically we can not change the connection parameters or the type of database system because the component only works as created.

.. code-block:: php

    <?php

    class SomeComponent
    {

        /**
         * The instantiation of the connection is hardcoded inside
         * the component so is difficult to replace it externally
         * or change its behavior
         */
        public function someDbTask()
        {
            $connection = new Connection(array(
                "host" => "localhost",
                "username" => "root",
                "password" => "secret",
                "dbname" => "invo"
            ));

            // ...
        }

    }

    $some = new SomeComponent();
    $some->someDbTask();

To solve this we create a setter that injects the dependency externally before use it. For now, this seems to be a good solution:

.. code-block:: php

    <?php

    class SomeComponent
    {

        protected $_connection;

        /**
         * Sets the connection externally
         */
        public function setConnection($connection)
        {
            $this->_connection = $connection;
        }

        public function someDbTask()
        {
            $connection = $this->_connection;

            // ...
        }

    }

    $some = new SomeComponent();

    //Create the connection
    $connection = new Connection(array(
        "host" => "localhost",
        "username" => "root",
        "password" => "secret",
        "dbname" => "invo"
    ));

    //Inject the connection in the component
    $some->setConnection($connection);

    $some->someDbTask();

Now consider that we use this component in different parts of the application, then we will need to create the connection several times before pass it to the component. This could be solved by using some kind of global registry where we obtain the connection instance and not have to create it again and again.

.. code-block:: php

    <?php

    class Registry
    {

        /**
         * Returns the connection
         */
        public static function getConnection()
        {
           return new Connection(array(
                "host" => "localhost",
                "username" => "root",
                "password" => "secret",
                "dbname" => "invo"
            ));
        }

    }

    class SomeComponent
    {

        protected $_connection;

        /**
         * Sets the connection externally
         */
        public function setConnection($connection){
            $this->_connection = $connection;
        }

        public function someDbTask()
        {
            $connection = $this->_connection;

            // ...
        }

    }

    $some = new SomeComponent();

    //Pass the connection defined in the registry
    $some->setConnection(Registry::getConnection());

    $some->someDbTask();

Now, let's imagine that we must to implement two methods in the component, the first always need to create a new connection and the second always need to use a shared connection:

.. code-block:: php

    <?php

    class Registry
    {

        protected static $_connection;

        /**
         * Creates a connection
         */
        protected static function _createConnection()
        {
            return new Connection(array(
                "host" => "localhost",
                "username" => "root",
                "password" => "secret",
                "dbname" => "invo"
            ));
        }

        /**
         * Creates a connection only once and returns it
         */
        public static function getSharedConnection()
        {
            if (self::$_connection===null){
                $connection = self::_createConnection();
                self::$_connection = $connection;
            }
            return self::$_connection;
        }

        /**
         * Always returns a new connection
         */
        public static function getNewConnection()
        {
            return self::_createConnection();
        }

    }

    class SomeComponent
    {

        protected $_connection;

        /**
         * Sets the connection externally
         */
        public function setConnection($connection){
            $this->_connection = $connection;
        }

        /**
         * This method always needs the shared connection
         */
        public function someDbTask()
        {
            $connection = $this->_connection;

            // ...
        }

        /**
         * This method always needs a new connection
         */
        public function someOtherDbTask($connection)
        {

        }

    }

    $some = new SomeComponent();

    //This injects the shared connection
    $some->setConnection(Registry::getSharedConnection());

    $some->someDbTask();

    //Here, we always pass a new connection as parameter
    $some->someOtherDbTask(Registry::getConnection());

So far we have seen how dependency injection solved our problems. Passing dependencies as arguments instead of creating them internally in the code makes our application more maintainable and decoupled. However to long term, this form of dependency injection have some disadvantages.

For instance, if the component has many dependencies, we will need to create multiple setter arguments to pass the dependencies or create a constructor that pass them with many arguments, additionally create dependencies before use the component, every time, makes our code not maintainable as we would like:

.. code-block:: php

    <?php

    //Create the dependencies or retrieve them from the registry
    $connection = new Connection();
    $session = new Session();
    $fileSystem = new FileSystem();
    $filter = new Filter();
    $selector = new Selector();

    //Pass them as constructor parameters
    $some = new SomeComponent($connection, $session, $fileSystem, $filter, $selector);

    // ... or using setters

    $some->setConnection($connection);
    $some->setSession($session);
    $some->setFileSystem($fileSystem);
    $some->setFilter($filter);
    $some->setSelector($selector);

Think we had to create this object in many parts of our application. If you ever do not require any of the dependencies, we need to go everywhere to remove the parameter in the constructor or the setter where we injected the code. To solve this we return again to a global registry to create the component. However, it adds a new layer of abstraction before creating the object:

.. code-block:: php

    <?php

    class SomeComponent
    {

        // ...

        /**
         * Define a factory method to create SomeComponent instances injecting its dependencies
         */
        public static function factory()
        {

            $connection = new Connection();
            $session = new Session();
            $fileSystem = new FileSystem();
            $filter = new Filter();
            $selector = new Selector();

            return new self($connection, $session, $fileSystem, $filter, $selector);
        }

    }

One moment, we returned back to the beginning, we are building again the dependencies inside the component! We can move on and find out a way to solve this problem every time. But it seems that time and again we fall back into bad practices.

A practical and elegant way to solve these problems is to use a container for dependencies. The containers act as the global registry that we saw earlier. Using the container for dependencies as a bridge to obtain the dependencies allows us to reduce the complexity of our component:

.. code-block:: php

    <?php

    class SomeComponent
    {

        protected $_di;

        public function __construct($di)
        {
            $this->_di = $di;
        }

        public function someDbTask()
        {

            // Get the connection service
            // Always returns a new connection
            $connection = $this->_di->get('db');

        }

        public function someOtherDbTask()
        {

            // Get a shared connection service,
            // this will return the same connection everytime
            $connection = $this->_di->getShared('db');

            //This method also requires a input filtering service
            $filter = $this->_db->get('filter');

        }

    }

    $di = new Phalcon\DI();

    //Register a "db" service in the container
    $di->set('db', function(){
        return new Connection(array(
            "host" => "localhost",
            "username" => "root",
            "password" => "secret",
            "dbname" => "invo"
        ));
    });

    //Register a "filter" service in the container
    $di->set('filter', function(){
        return new Filter();
    });

    //Register a "session" service in the container
    $di->set('session', function(){
        return new Session();
    });

    //Pass the service container as unique parameter
    $some = new SomeComponent($di);

    $some->someTask();

The component now simply access the service it require when it needs it, if it does not requires a service, that is not even initialized saving resources. The component is now highly decoupled. For example, we can replace the manner in which connections are created, their behavior or any other aspect of them and that would not affect the component.

Our approach
------------

Phalcon\\DI is a component that implements Dependency Injection of services and it's itself a container for them.

Since Phalcon is highly decoupled, Phalcon\\DI is essential to integrate the different components of the framework. The developer can also use this component to inject dependencies and manage global instances of the different classes used in the application.

Basically, this component implements the `Inversion of Control`_ pattern. Applying this, the objects do not receive their dependencies using setters or constructors, but requesting a service dependency injector. This reduces the overall complexity, since there is only one way to get the required dependencies within a component.

Additionally, this pattern increases testability in the code, thus making it less prone to errors.

Registering services in the Container
-------------------------------------
Services can be registered by the framework itself or the developer. When a component A requires component B (or an instance of its class) to operate, it can request component B from the container, rather than creating a new instance component B.

This way of working gives us many advantages:

* We can replace a component by one created by ourselves or a third party one easily.
* We have full control of the object initialization, allowing us to set this objects as you need before delivery them to components.
* We can get global instances of components in a structured and unified way

Services can be registered in several ways:

.. code-block:: php

    <?php

	//Create the Dependency Injector Container
	$di = new Phalcon\DI();

	//By its class name
	$di->set("request", 'Phalcon\Http\Request');

	//Using an anonymous function, the instance will lazy loaded
	$di->set("request", function(){
	    return new Phalcon\Http\Request();
	});

	//Registering directly an instance
	$di->set("request", new Phalcon\Http\Request());

	//Using an array definition
	$di->set("request", array(
	    "className" => 'Phalcon\Http\Request'
	));

In the above example, when the framework needs to access the request data, it will ask for the service identified as ‘request’ in the container. The container in turn will return an instance of the required service. A developer might eventually replace a component when he/she needs.

Each of the methods (demonstrated in the above example) used to set/register a service has advantages and disadvantages. It is up to the developer and the particular requirements that will designate which one is used.

Setting a service by a string is simple but lacks flexibility. Setting services using an array offers a lot more flexibility but makes the code more complicated. The lambda function is a good balance between the two but could lead to more maintenance than one would expect.

Phalcon\\DI offers lazy loading for every service it stores. Unless the developer chooses to instantiate an object directly and store it in the container, any object stored in it (via array, string etc.) will be lazy loaded i.e. instantiated only when requested.

.. code-block:: php

    <?php

    //Register a service "db" with a class name and its parameters
    $di->set("db", array(
        "className" => "Phalcon\Db\Adapter\Pdo\Mysql",
        "parameters" => array(
              "parameter" => array(
                   "host" => "localhost",
                   "username" => "root",
                   "password" => "secret",
                   "dbname" => "blog"
              )
        )
    ));

    //Using an anonymous function
    $di->set("db", function(){
        return new Phalcon\Db\Adapter\Pdo\Mysql(array(
             "host" => "localhost",
             "username" => "root",
             "password" => "secret",
             "dbname" => "blog"
        ));
    });

Both service registrations above produce the same result. The array definition however, allows for alteration of the service parameters if needed:

.. code-block:: php

    <?php

    $di->setParameter("db", 0, array(
        "host" => "localhost",
        "username" => "root",
        "password" => "secret"
    ));

Obtaining a service from the container is a matter of simply calling the “get” method. A new instance of the service will be returned:

.. code-block:: php

    <?php $request = $di->get("request");

Or by calling through the magic method:

.. code-block:: php

    <?php

    $request = $di->getRequest();

Phalcon\\DI also allows for services to be reusable. To get a service previously instantiated the getShared() method can be used. Specifically for the Phalcon\\Http\\Request example shown above:

.. code-block:: php

    <?php

    $request = $di->getShared("request");

Arguments can be passed to the constructor by adding an array parameter to the method "get":

.. code-block:: php

    <?php

    $component = $di->get("MyComponent", array("some-parameter", "other"))

Factory Default DI
------------------
Although the decoupled character of Phalcon offers us great freedom and flexibility, maybe we just simply want to use it as a full-stack framework. To achieve this, the framework provides a variant of Phalcon\\DI called Phalcon\\DI\\FactoryDefault. This class automatically registers the appropriate services bundled with the framework to act as full-stack.

.. code-block:: php

    <?php $di = new Phalcon\DI\FactoryDefault();

Service Name Conventions
------------------------
Although you can register services with the names you want. Phalcon has a seriers of service naming conventions that allow it to get the right services when you need it requires them.

+----------------+---------------------------------------------+-----------------------------------------------------------------------------------------+
| Service Name   | Description                                 | Default                                                                                 |
+================+=============================================+=========================================================================================+
| dispatcher     | Controllers Dispatching Service             | :doc:`Phalcon\\Mvc\\Dispatcher <../api/Phalcon_Mvc_Dispatcher>`                         |
+----------------+---------------------------------------------+-----------------------------------------------------------------------------------------+
| router         | Routing Service                             | :doc:`Phalcon\\Mvc\\Router <../api/Phalcon_Mvc_Router>`                                 |
+----------------+---------------------------------------------+-----------------------------------------------------------------------------------------+
| url            | URL Generator Service                       | :doc:`Phalcon\\Mvc\\Url <../api/Phalcon_Mvc_Url>`                                       |
+----------------+---------------------------------------------+-----------------------------------------------------------------------------------------+
| request        | HTTP Request Environment Service            | :doc:`Phalcon\\Http\\Request <../api/Phalcon_Http_Request>`                             |
+----------------+---------------------------------------------+-----------------------------------------------------------------------------------------+
| response       | HTTP Response Environment Service           | :doc:`Phalcon\\Http\\Response <../api/Phalcon_Http_Response>`                           |
+----------------+---------------------------------------------+-----------------------------------------------------------------------------------------+
| filter         | Input Filtering Service                     | :doc:`Phalcon\\Filter <../api/Phalcon_Filter>`                                          |
+----------------+---------------------------------------------+-----------------------------------------------------------------------------------------+
| flash          | Flash Messaging Service                     | :doc:`Phalcon\\Flash\\Direct <../api/Phalcon_Flash_Direct>`                             |
+----------------+---------------------------------------------+-----------------------------------------------------------------------------------------+
| session        | Session Service                             | :doc:`Phalcon\\Session\\Adapter\\Files <../api/Phalcon_Session_Adapter_Files>`          |
+----------------+---------------------------------------------+-----------------------------------------------------------------------------------------+
| eventsManager  | Events Management Service                   | :doc:`Phalcon\\Events\\Manager <../api/Phalcon_Events_Manager>`                         |
+----------------+---------------------------------------------+-----------------------------------------------------------------------------------------+
| db             | Low-Level Database Connection Service       | :doc:`Phalcon\\Db <../api/Phalcon_Db>`                                                  |
+----------------+---------------------------------------------+-----------------------------------------------------------------------------------------+
| modelsManager  | Models Management Service                   | :doc:`Phalcon\\Mvc\\Model\\Manager <../api/Phalcon_Mvc_Model_Manager>`                  |
+----------------+---------------------------------------------+-----------------------------------------------------------------------------------------+
| modelsMetadata | Models Meta-Data Service                    | :doc:`Phalcon\\Mvc\\Model\\MetaData\\Memory <../api/Phalcon_Mvc_Model_MetaData_Memory>` |
+----------------+---------------------------------------------+-----------------------------------------------------------------------------------------+

Instantiating classes via the Services Container
------------------------------------------------
When you request a service to the services container, if it can't find out a service with the same name it'll try to load a class with the same name. With this behavior we can replace any class by another simply by registering a service with its name:

.. code-block:: php

    <?php

    //Register a controller as a service
    $di->set('IndexController', function() {
        $component = new Component();
        return $component;
    });

    //Register a controller as a service
    $di->set('MyOtherComponent', function() {
        //Actually returns another component
        $component = new AnotherComponent();
        return $component;
    });

    //Create a instance via the services container
    $myComponent = $di->get('MyOtherComponent');

You can take advantage of this, always instantiating your classes via the services container (even if they aren't registered as services). The DI will fallback to a valid autoloader to finally load the class.


.. _`Inversion of Control`: http://en.wikipedia.org/wiki/Inversion_of_control