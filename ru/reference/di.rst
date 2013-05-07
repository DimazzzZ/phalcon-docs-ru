Использование Dependency Injection
**********************************
Следующий пример немного длинный, но объясняет использование контейнера сервисов (service container), service location и dependency injection. Итак, представим, что мы разрабатываем компонент, назовём его SomeComponent. Сейчас нам не важно, какую именно задачу он выполняет. Наш компонент имеет некоторую зависимость, отвечающую за соединение с базой данных.

В первом примере соединение устанавливается внутри компонента. Такой подход не является практичным, он не позволяет нам изменить параметры соединения или тип СУБД, потому как компонент работает только так, как был создан.

.. code-block:: php

    <?php

    class SomeComponent
    {

        /**
         * Объект соединения жестко вписан в компонент,
         * что усложняет его замену на какой-то
         * внешний или изменение его поведения
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

Чтобы решить эту проблему, создадим сеттер (setter), который внедрит внешнюю зависимость перед использованием. Теперь это похоже на хорошее решение:

.. code-block:: php

    <?php

    class SomeComponent
    {

        protected $_connection;

        /**
         * Назначает внешнее соединение
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

    // Создание соединения с БД
    $connection = new Connection(array(
        "host" => "localhost",
        "username" => "root",
        "password" => "secret",
        "dbname" => "invo"
    ));

    // Внедрение соединения в компонент
    $some->setConnection($connection);

    $some->someDbTask();

Теперь примем во внимание тот факт, что мы используем компонент в различных частях приложения, поэтому появляется необходимость создавать соединение несколько раз и передавать его в компонент. С помощью некоторого глобального регистра будем получать копию соединения, тем самым нам больше нет надобности создавать его вновь и вновь:

.. code-block:: php

    <?php

    class Registry
    {

        /**
         * Возвращает соединение
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
         * Назначает внешнее соединение
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

    // Передает соединение, определяемое в регистре
    $some->setConnection(Registry::getConnection());

    $some->someDbTask();

Теперь представим, что нам необходимо реализовать в компоненте два метода: первый всегда нуждается в создании нового соединения, а второй всегда использует уже установленное (shared):

.. code-block:: php

    <?php

    class Registry
    {

        protected static $_connection;

        /**
         * Создаёт соединение
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
         * Создаёт соединение единожды и возвращает его
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
         * Всегда возвращает новое соединение
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
         * Назначает внешнее соединение
         */
        public function setConnection($connection){
            $this->_connection = $connection;
        }

        /**
         * Для этого метода всегда требуется уже установленное соединение
         */
        public function someDbTask()
        {
            $connection = $this->_connection;

            // ...
        }

        /**
         * Для этого метода всегда требуется новое соединение
         */
        public function someOtherDbTask($connection)
        {

        }

    }

    $some = new SomeComponent();

    // Тут внедряется уже установленное (shared) соединение
    $some->setConnection(Registry::getSharedConnection());

    $some->someDbTask();

    // А здесь всегда в качестве параметра передаётся новое соединение
    $some->someOtherDbTask(Registry::getConnection());

До сих пор мы рассматривали случаи, когда внедрение зависимостей решает наши задачи. Передача зависимости в качестве аргументов вместо создания их внутри кода делает наше приложение более гибким и уменьшает его связанность. Однако, в перспективе, такая форма внедрения зависимостей имеет некоторые недостатки.

Например, если компонент имеет много зависимостей, мы будем вынуждены создавать сеттеры с множеством аргументов для передачи зависимостей или конструктор, который принимает их в качестве большого числа аргументов, вдобавок к этому, всякий раз создавать ещё и сами зависимости до использования компонента. Это сделает наш код слишком сложным для сопровождения:

.. code-block:: php

    <?php

    // Создание зависимостей или получение их из регистра
    $connection = new Connection();
    $session = new Session();
    $fileSystem = new FileSystem();
    $filter = new Filter();
    $selector = new Selector();

    // Передача их в конструктор в качестве параметров
    $some = new SomeComponent($connection, $session, $fileSystem, $filter, $selector);

    // ... или использование сеттеров

    $some->setConnection($connection);
    $some->setSession($session);
    $some->setFileSystem($fileSystem);
    $some->setFilter($filter);
    $some->setSelector($selector);

Думаю, пришлось бы создавать этот объект во многих частях нашего приложения. Если когда-нибудь мы перестанем нуждаться в какой-либо зависимости, нам придётся пройтись по всем этим местам и удалить соответствующий параметр в вызовах конструктора или сеттерах. Чтобы решить эту проблему, вернёмся к глобальному регистру для создания компонента. Однако, это добавит новый уровень обстракции, предшествующий созданию объекта:

.. code-block:: php

    <?php

    class SomeComponent
    {

        // ...

        /**
         * Определение метода factory, который создаёт экземпляр SomeComponent и внедряет в него зависимости
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

Минуточку, мы снова вернулись туда, откуда начали: создание зависимостей внутри компонента! Мы можем двигаться дальше и находить способ решать эту проблему каждый раз. Но, это означает, что мы снова и снова будем наступать на те же грабли.

Практически применимый и элегантный способ решить эту проблему — это использовать контейнер для зависимостей. Он играет ту же роль, что и глобальный регистр, который мы видели выше. Использование контейнера в качестве моста к зависимостям позволяет нам уменьшить сложность нашего компонента:

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

            // Получение сервиса соединений
            // Всегда возвращает соединение
            $connection = $this->_di->get('db');

        }

        public function someOtherDbTask()
        {

            // Получение сервиса соединения, предназначенного для общего доступа,
            // всегда возвращает одно и то же соединение
            $connection = $this->_di->getShared('db');

            // Этот метод так же требует сервиса фильтрации входных данных
            $filter = $this->_db->get('filter');

        }

    }

    $di = new Phalcon\DI();

    // Регистрация в контейнере сервиса "db"
    $di->set('db', function() {
        return new Connection(array(
            "host" => "localhost",
            "username" => "root",
            "password" => "secret",
            "dbname" => "invo"
        ));
    });

    // Регистрация в контейнере сервиса "filter"
    $di->set('filter', function() {
        return new Filter();
    });

    // Регистрация в контейнере сервиса "session"
    $di->set('session', function() {
        return new Session();
    });

    // Передача контейнера сервисов в качестве единственного параметра
    $some = new SomeComponent($di);

    $some->someTask();

Теперь компонент имеет простой доступ к сервисам, которые ему необходимы. Если сервис не востребован, он не будет инициализирован, тем самым экономя ресурсы. Так же компонент теперь обладает низкой связанностью. Например, можно заменить способ создания соединений, поведение или любой другой аспект их работы, и это никак не отразится на компоненте.

Наш подход
==========
Phalcon\\DI — это компонент, реализующий Dependency Injection и Location сервисов и является контейнером для них.

Поскольку Phalcon обладает низкой связанностью, Phalcon\\DI необходимо обеспечить интеграцию различных компонентов фреймворка. Разработчики так же могут использовать этот компонент для внедрения зависимостей и использования глобальных экземпляров различных классов, используемых в приложении.

В основе своей, компонент реализует паттерн `Инверсии управления`_. Применяя его, объекты получают их зависимости не с использованием сеттеров или конструкторов, а с помощью сервиса внедрения зависимостей. Это снижает общую сложность, поскольку остаётся только один способ получения зависимостей в компоненте.

К тому же, этот паттерн увеличивает тестируемость в коде, что позволяет снизить "ошибочность" кода.

Регистрация сервисов в Контейнере сервисов
==========================================
Регистрация сервисов возможна как разработчиком, так и самим фреймворком. Когда компоненту A требуется компонент B (или экземпляр его класса) для работы, он может запросить его из контейнера, а не создавать новый экземпляр.

Такой способ работы даёт нам много преимуществ: 

* Мы можем легко заменять компонент на созданный нами или кем-то другим.
* Мы обладаем полным контролем над инициализацией объекта, что позволяет нам настраивать эти объекты так, как нам необходимо, прежде, чем передать их компонентам.
* Мы можем получать глобальный экземпляр компонента структурированным и унифицированным образом.

Зарегистрировать сервисы можно несколькими различными способами:

.. code-block:: php

    <?php

    // Создание контейнера DI
    $di = new Phalcon\DI();

    // По названию класса
    $di->set("request", 'Phalcon\Http\Request');

    // С использованием анонимной функции для отложенной загрузки
    $di->set("request", function() {
        return new Phalcon\Http\Request();
    });

    // Регистрация экземпляра напрямую
    $di->set("request", new Phalcon\Http\Request());

    // Определение с помощью массива
    $di->set("request", array(
        "className" => 'Phalcon\Http\Request'
    ));

Для регистрации сервисов можно так же использовать синтаксис массивов:
The array syntax is also allowed to register services:

.. code-block:: php

    <?php

    // Создание контейнера DI
    $di = new Phalcon\DI();

    // По названию класса
    $di["request"] = 'Phalcon\Http\Request';

    // С использованием анонимной функции для отложенной загрузки
    $di["request"] = function() {
        return new Phalcon\Http\Request();
    };

    // Регистрация экземпляра напрямую
    $di["request"] = new Phalcon\Http\Request();

    // Определение с помощью массива
    $di["request"] = array(
        "className" => 'Phalcon\Http\Request'
    );

В примере, данном выше, когда фреймворк нуждается в доступе к запрашиваемым данным, он будет запрашивать в контейнере сервис, названный 'request'.
Контейнер, в свою очередь, возвращает экземпляр затребованного сервиса. Разработчик, в конечном итоге, может заменить компонент, когда захочет.

Каждый из методов регистрации сервисов имеет свои достоинства и недостатки. Какой из них использовать — зависит только от разработчика и от конкретных требований.

Назначение сервиса строкой очень простое, но лишено гибкости. В качестве массива — предоставляет большую гибкость, но делает код менее понятным. Анонимные функции неплохо балансируют между этими двумя способами, но им может потребоваться больше обслуживания, чем это ожидается.

Phalcon\\DI предоставляет отложенную загрузку для каждого хранимого им сервиса. Если разработчик не решит создавать экземпляр объекта напрямую и хранить его в контейнере, любой объект будет сохранённый в нём (через массив, строку и т.д.) будет загружен отложенно (lazy load), т.е. создастся только тогда, когда будет востребован.

Простая регистрация
-------------------

Как мы было показано выше, есть несколько способов для регистрации сервисов. Следующие из них мы называем "простыми":

Строчный
^^^^^^^^
Этот способ ожидает в качестве параметра имя существующего класса, возвращает его объект, если класс не был загружен автолоадером.
Такой способ не позволяет передавать аргументы для конструктора класса или настраивать параметры:

.. code-block:: php

    <?php

    // Возвращает новый Phalcon\Http\Request();
    $di->set('request', 'Phalcon\Http\Request');

Объект
^^^^^^
Этот способ в качестве параметра принимает объект. Объект не нуждается в создании, потому как объект уже является объектом сам по себе. Вообще говоря, в данном случае это не является настоящим внедрением зависимости, однако такой способ вполне используем, если вы хотите быть уверены в том, что возвращаемая зависимость всегда будет одним и тем же объектом/значением:

.. code-block:: php

    <?php

    // Возвращает новый Phalcon\Http\Request();
    $di->set('request', new Phalcon\Http\Request());

Замыкания/Анонимные функции
^^^^^^^^^^^^^^^^^^^^^^^^^^^
Этот метод дает больше свободы для построения зависимости, если этого захотеть, тем не менее, он весьма сложен в плане изменения некоторых параметров извне без полного замещения определения зависимости:

.. code-block:: php

    <?php

    $di->set("db", function() {
        return new \Phalcon\Db\Adapter\Pdo\Mysql(array(
             "host" => "localhost",
             "username" => "root",
             "password" => "secret",
             "dbname" => "blog"
        ));
    });

Некоторые ограничения можно преодолеть путём передачи дополнительных переменных в область видимости замыкания:

.. code-block:: php

    <?php

    // Использование переменной $config в текущей области видимости
    $di->set("db", function() use ($config) {
        return new \Phalcon\Db\Adapter\Pdo\Mysql(array(
             "host" => $config->host,
             "username" => $config->username,
             "password" => $config->password,
             "dbname" => $config->name
        ));
    });

Сложная регистрация
-------------------
Если потребуется изменить определение сервиса без создания экземпляра, тогда нам придётся определять его с использованием синтаксиса массивов. Такое определение может оказаться чуть более длинным:

.. code-block:: php

    <?php

    // Регистрация сервиса 'logger' с помощью имени класса и параметров для него
    $di->set('logger', array(
        'className' => 'Phalcon\Logger\Adapter\File',
        'arguments' => array(
            array(
                'type' => 'parameter',
                'value' => '../apps/logs/error.log'
            )
        )
    ));

    // Или в виде анонимной функции
    $di->set('logger', function() {
        return new \Phalcon\Logger\Adapter\File('../apps/logs/error.log');
    });

Оба способа приведут к одинаковому результату. Определение же с помощью массива позволяет изменение параметров, если это неободимо:

.. code-block:: php

    <?php

    // Измнение названия класса для сервиса
    $di->getService('logger')->setClassName('MyCustomLogger');

    // Измнение первого параметра без пересоздания экземпляра сервиса logger
    $di->getService('logger')->setParameter(0, array(
        'type' => 'parameter',
        'value' => '../apps/logs/error.log'
    ));

В дополнение к этому, используя синтаксис массивов, можно использовать три типа внедрения зависимостей:

Constructor Injection
^^^^^^^^^^^^^^^^^^^^^
Этот тип передаёт зависимости/аргументы в конструктор класса.
Представим, что у нас есть следующий компонент:

.. code-block:: php

    <?php

    namespace SomeApp;

    use Phalcon\Http\Response;

    class SomeComponent
    {

        protected $_response;

        protected $_someFlag;

        public function __construct(Response $response, $someFlag)
        {
            $this->_response = $response;
            $this->_someFlag = $someFlag;
        }

    }

Сервис может быть зарегистрирован следующим образом:

.. code-block:: php

    <?php

    $di->set('response', array(
        'className' => 'Phalcon\Http\Response'
    ));

    $di->set('someComponent', array(
        'className' => 'SomeApp\SomeComponent',
        'arguments' => array(
            array('type' => 'service', 'name' => 'response'),
            array('type' => 'parameter', 'value' => true)
        )
    ));

Сервис "response" (Phalcon\\Http\\Response) передаётся в конструктор в качестве первого параметра, в то время как вторым параметром передаётся булевое значение (true) без изменений.

Setter Injection
^^^^^^^^^^^^^^^^
Классы могут иметь сеттеры для внедрения дополнительных зависимостей. Наш предыдущий класс может быть изменён, чтобы принимать зависимости с помощью сеттеров:

.. code-block:: php

    <?php

    namespace SomeApp;

    use Phalcon\Http\Response;

    class SomeComponent
    {

        protected $_response;

        protected $_someFlag;

        public function setResponse(Response $response)
        {
            $this->_response = $response;
        }

        public function setFlag($someFlag)
        {
            $this->_someFlag = $someFlag;
        }

    }

Сервис с сеттерами для зависимостей может быть зарегистрирован следующим образом:

.. code-block:: php

    <?php

    $di->set('response', array(
        'className' => 'Phalcon\Http\Response'
    ));

    $di->set('someComponent', array(
        'className' => 'SomeApp\SomeComponent',
        'calls' => array(
            array(
                'method' => 'setResponse',
                'arguments' => array(
                    array('type' => 'service', 'name' => 'response'),
                )
            ),
            array(
                'method' => 'setFlag',
                'arguments' => array(
                    array('type' => 'parameter', 'value' => true)
                )
            )
        )
    ));

Properties Injection
^^^^^^^^^^^^^^^^^^^^
Менее распространённым способом является внедрение зависимостей или полей класса напрямую:

.. code-block:: php

    <?php

    namespace SomeApp;

    use Phalcon\Http\Response;

    class SomeComponent
    {

        public $response;

        public $someFlag;

    }

Сервис с прямым внедрением может быть зарегистрирован следующим способом:

.. code-block:: php

    <?php

    $di->set('response', array(
        'className' => 'Phalcon\Http\Response'
    ));

    $di->set('someComponent', array(
        'className' => 'SomeApp\SomeComponent',
        'properties' => array(
            array(
                'name' => 'response',
                'value' => array('type' => 'service', 'name' => 'response')
            ),
            array(
                'name' => 'someFlag',
                'value' => array('type' => 'parameter', 'value' => true)
            )
        )
    ));

Поддерживаются параметры следующих типов:

+-------------+----------------------------------------------------------+-------------------------------------------------------------------------------------+
| Type        | Description                                              | Example                                                                             |
+=============+==========================================================+=====================================================================================+
| parameter   | Represents a literal value to be passed as parameter     | array('type' => 'parameter', 'value' => 1234)                                       |
+-------------+----------------------------------------------------------+-------------------------------------------------------------------------------------+
| service     | Represents another service in the services container     | array('type' => 'service', 'name' => 'request')                                     |
+-------------+----------------------------------------------------------+-------------------------------------------------------------------------------------+
| instance    | Represents an object that must be built dynamically      | array('type' => 'instance', 'className' => 'DateTime', 'arguments' => array('now')) |
+-------------+----------------------------------------------------------+-------------------------------------------------------------------------------------+

Resolving a service whose definition is complex may be slightly slower than previously seen simple definitions. However,
these provide a more robust approach to define and inject services.

Mixing different types of definitions is allowed, everyone can decide what is the most appropriate way to register the services
according to the application needs.

Resolving Services
==================
Obtaining a service from the container is a matter of simply calling the “get” method. A new instance of the service will be returned:

.. code-block:: php

    <?php $request = $di->get("request");

Or by calling through the magic method:

.. code-block:: php

    <?php

    $request = $di->getRequest();

Or using the array-access syntax:

.. code-block:: php

    <?php

    $request = $di['request'];

Arguments can be passed to the constructor by adding an array parameter to the method "get":

.. code-block:: php

    <?php

    // new MyComponent("some-parameter", "other")
    $component = $di->get("MyComponent", array("some-parameter", "other"));

Shared services
===============
Services can be registered as "shared" services this means that they always will act as singletons_. Once the service is resolved for the first time
the same instance it's returned every time a consumer retrieve the service from the container:

.. code-block:: php

    <?php

    //Register the session service as "always shared"
    $di->setShared('session', function() {
        $session = new Phalcon\Session\Adapter\Files();
        $session->start();
        return $session;
    });

    $session = $di->get('session'); // Locates the service for the first time
    $session = $di->getSession(); // Returns the first instantiated object

An alternative way to register services is pass "true" as third parameter of "set":

.. code-block:: php

    <?php

    //Register the session service as "always shared"
    $di->set('session', function() {
        //...
    }, true);

If a service isn't registered as shared and you want to be sure that a shared instance will be accessed every time
the service is obtained from the DI, you can use the 'getShared' method:

.. code-block:: php

    <?php

    $request = $di->getShared("request");

Manipulating services individually
==================================
Once a service is registered in services container, you can retrieve it to manipulate it individually:

.. code-block:: php

    <?php

    //Register the session service as "always shared"
    $di->set('request', 'Phalcon\Http\Request');

    //Get the service
    $requestService = $di->getService('request');

    //Change its definition
    $requestService->setDefinition(function() {
        return new Phalcon\Http\Request();
    });

    //Change it to shared
    $request->setShared(true);

    //Resolve the service (return a Phalcon\Http\Request instance)
    $request = $requestService->resolve();

Instantiating classes via the Services Container
================================================
When you request a service to the services container, if it can't find out a service with the same name it'll try to load a class with
the same name. With this behavior we can replace any class by another simply by registering a service with its name:

.. code-block:: php

    <?php

    //Register a controller as a service
    $di->set('IndexController', function() {
        $component = new Component();
        return $component;
    }, true);

    //Register a controller as a service
    $di->set('MyOtherComponent', function() {
        //Actually returns another component
        $component = new AnotherComponent();
        return $component;
    });

    //Create a instance via the services container
    $myComponent = $di->get('MyOtherComponent');

You can take advantage of this, always instantiating your classes via the services container (even if they aren't registered as services). The DI will
fallback to a valid autoloader to finally load the class. By doing this, you can easily replace any class in the future by implementing a definition
for it.

Automatic Injecting of the DI itself
====================================
If a class or component requires the DI itself to locate services, the DI can automatically inject itself to the instances creates by it,
to do this, you need to implement the :doc:`Phalcon\\DI\\InjectionAwareInterface <../api/Phalcon_DI_InjectionAwareInterface>` in your classes:

.. code-block:: php

    <?php

    class MyClass implements \Phalcon\DI\InjectionAwareInterface
    {

        protected $_di;

        public function setDi($di)
        {
            $this->_di = $di;
        }

        public function getDi()
        {
            return $this->_di;
        }

    }

Then once the service is resolved, the $di will be passed to setDi automatically:

.. code-block:: php

    <?php

    //Register the service
    $di->set('myClass', 'MyClass');

    //Resolve the service (also $myClass->setDi($di) is automatically called)
    $myClass = $di->get('myClass');

Avoiding service resolution
===========================
Some services are used in each of the requests made to the application, eliminate the process of resolving the service
could add some small improvement in performance.

.. code-block:: php

    <?php

    //Resolve the object externally instead of using a definition for it:
    $router = new MyRouter();

    //Pass the resolved object to the service registration
    $di->set('router', $router);

Organizing services in files
============================
You can better organize your application by moving the service registration to individual files instead of
doing everything in the application's bootstrap:

.. code-block:: php

    <?php

    $di->set('router', function() {
        return include ("../app/config/routes.php");
    });

Then in the file ("../app/config/routes.php") return the object resolved:

.. code-block:: php

    <?php

    $router = new MyRouter();

    $router->post('/login');

    return $router;

Accessing the DI in a static way
================================
If needed you can access the latest DI created in a static function in the following way:

.. code-block:: php

    <?php

    class SomeComponent
    {

        public static function someMethod()
        {
            //Get the session service
            $session = Phalcon\DI::getDefault()->getSession();
        }

    }

Factory Default DI
==================
Although the decoupled character of Phalcon offers us great freedom and flexibility, maybe we just simply want to use it as a full-stack
framework. To achieve this, the framework provides a variant of Phalcon\\DI called Phalcon\\DI\\FactoryDefault. This class automatically
registers the appropriate services bundled with the framework to act as full-stack.

.. code-block:: php

    <?php $di = new Phalcon\DI\FactoryDefault();

Service Name Conventions
========================
Although you can register services with the names you want. Phalcon has a seriers of service naming conventions that allow it to get the
right services when you need it requires them.

+---------------------+---------------------------------------------+----------------------------------------------------------------------------------------------------+--------+
| Service Name        | Description                                 | Default                                                                                            | Shared |
+=====================+=============================================+====================================================================================================+========+
| dispatcher          | Controllers Dispatching Service             | :doc:`Phalcon\\Mvc\\Dispatcher <../api/Phalcon_Mvc_Dispatcher>`                                    | Yes    |
+---------------------+---------------------------------------------+----------------------------------------------------------------------------------------------------+--------+
| router              | Routing Service                             | :doc:`Phalcon\\Mvc\\Router <../api/Phalcon_Mvc_Router>`                                            | Yes    |
+---------------------+---------------------------------------------+----------------------------------------------------------------------------------------------------+--------+
| url                 | URL Generator Service                       | :doc:`Phalcon\\Mvc\\Url <../api/Phalcon_Mvc_Url>`                                                  | Yes    |
+---------------------+---------------------------------------------+----------------------------------------------------------------------------------------------------+--------+
| request             | HTTP Request Environment Service            | :doc:`Phalcon\\Http\\Request <../api/Phalcon_Http_Request>`                                        | Yes    |
+---------------------+---------------------------------------------+----------------------------------------------------------------------------------------------------+--------+
| response            | HTTP Response Environment Service           | :doc:`Phalcon\\Http\\Response <../api/Phalcon_Http_Response>`                                      | Yes    |
+---------------------+---------------------------------------------+----------------------------------------------------------------------------------------------------+--------+
| filter              | Input Filtering Service                     | :doc:`Phalcon\\Filter <../api/Phalcon_Filter>`                                                     | Yes    |
+---------------------+---------------------------------------------+----------------------------------------------------------------------------------------------------+--------+
| flash               | Flash Messaging Service                     | :doc:`Phalcon\\Flash\\Direct <../api/Phalcon_Flash_Direct>`                                        | Yes    |
+---------------------+---------------------------------------------+----------------------------------------------------------------------------------------------------+--------+
| flashSession        | Flash Session Messaging Service             | :doc:`Phalcon\\Flash\\Session <../api/Phalcon_Flash_Session>`                                      | Yes    |
+---------------------+---------------------------------------------+----------------------------------------------------------------------------------------------------+--------+
| session             | Session Service                             | :doc:`Phalcon\\Session\\Adapter\\Files <../api/Phalcon_Session_Adapter_Files>`                     | Yes    |
+---------------------+---------------------------------------------+----------------------------------------------------------------------------------------------------+--------+
| eventsManager       | Events Management Service                   | :doc:`Phalcon\\Events\\Manager <../api/Phalcon_Events_Manager>`                                    | Yes    |
+---------------------+---------------------------------------------+----------------------------------------------------------------------------------------------------+--------+
| db                  | Low-Level Database Connection Service       | :doc:`Phalcon\\Db <../api/Phalcon_Db>`                                                             | Yes    |
+---------------------+---------------------------------------------+----------------------------------------------------------------------------------------------------+--------+
| security            | Security helpers                            | :doc:`Phalcon\\Security <../api/Phalcon_Security>`                                                 | Yes    |
+---------------------+---------------------------------------------+----------------------------------------------------------------------------------------------------+--------+
| escaper             | Contextual Escaping                         | :doc:`Phalcon\\Escaper <../api/Phalcon_Escaper>`                                                   | Yes    |
+---------------------+---------------------------------------------+----------------------------------------------------------------------------------------------------+--------+
| annotations         | Annotations Parser                          | :doc:`Phalcon\\Annotations\\Adapter\\Memory <../api/Phalcon_Annotations_Adapter_Memory>`           | Yes    |
+---------------------+---------------------------------------------+----------------------------------------------------------------------------------------------------+--------+
| modelsManager       | Models Management Service                   | :doc:`Phalcon\\Mvc\\Model\\Manager <../api/Phalcon_Mvc_Model_Manager>`                             | Yes    |
+---------------------+---------------------------------------------+----------------------------------------------------------------------------------------------------+--------+
| modelsMetadata      | Models Meta-Data Service                    | :doc:`Phalcon\\Mvc\\Model\\MetaData\\Memory <../api/Phalcon_Mvc_Model_MetaData_Memory>`            | Yes    |
+---------------------+---------------------------------------------+----------------------------------------------------------------------------------------------------+--------+
| transactionManager  | Models Transaction Manager Service          | :doc:`Phalcon\\Mvc\\Model\\Transaction\\Manager <../api/Phalcon_Mvc_Model_Transaction_Manager>`    | Yes    |
+---------------------+---------------------------------------------+----------------------------------------------------------------------------------------------------+--------+
| modelsCache         | Cache backend for models cache              | None                                                                                               | -      |
+---------------------+---------------------------------------------+----------------------------------------------------------------------------------------------------+--------+
| viewsCache          | Cache backend for views fragments           | None                                                                                               | -      |
+---------------------+---------------------------------------------+----------------------------------------------------------------------------------------------------+--------+

Implementing your own DI
========================
The :doc:`Phalcon\\DiInterface <../api/Phalcon_DiInterface>` interface must be implemented to create your own DI replacing the one provided by Phalcon or extend the current one.

.. _`Инверсии управления`: http://en.wikipedia.org/wiki/Inversion_of_control
.. _Singletons: http://en.wikipedia.org/wiki/Singleton_pattern
