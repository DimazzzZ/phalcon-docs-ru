Микроприложения
===============
С помощью Phalcon можно создавать приложения по типу "Микрофреймворк".
Для этого, необходимо написать всего лишь несколько строк кода. Микроприложения подходят для реализации
небольших приложений, различныx API и прототипов на практике.

.. code-block:: php

    <?php

    $app = new Phalcon\Mvc\Micro();

    $app->get('/say/welcome/{name}', function ($name) {
        echo "<h1>Welcome $name!</h1>";
    });

    $app->handle();

Создание микроприложения
------------------------
:doc:`Phalcon\\Mvc\\Micro <../api/Phalcon_Mvc_Micro>` это класс, отвечающий за реализацию микроприложения.

.. code-block:: php

    <?php

    $app = new Phalcon\Mvc\Micro();

Создание путей
--------------
После создания экземпляра класса необходимо добавить некоторые пути. :doc:`Phalcon\\Mvc\\Router <../api/Phalcon_Mvc_Router>` 
отвечает за управление путями, которые должны всегда начинаться с  /. При создании путей необходимо указывать, какой метод 
HTTP используется, чтобы запросы путей соответствовали методам HTTP. Ниже представлен пример, показывающий как создавать пути 
используя метод GET:

.. code-block:: php

    <?php

    $app->get('/say/hello/{name}', function ($name) {
        echo "<h1>Hello! $name</h1>";
    });

Метод "get" показывает, что используется GET-запрос. Путь /say/hello/{name} также имеет параметр {$name},
который напрямую передается обработчику пути (анонимная функция). Обработка пути выполняется, когда путь совпадает.
Обработчик может быть любого типа, который возвращает данные в PHP-среде. Следующий пример демонстрирует,
как создавать различные типы обработчиков пути:

.. code-block:: php

    <?php

    // С помощью функции
    function say_hello($name) {
        echo "<h1>Hello! $name</h1>";
    }

    $app->get('/say/hello/{name}', "say_hello");

    // С помощью статического метода
    $app->get('/say/hello/{name}', "SomeClass::someSayMethod");

    // С помощью метода объекта
    $myController = new MyController();
    $app->get('/say/hello/{name}', array($myController, "someAction"));

    // Анонимная функция (замыкание)
    $app->get('/say/hello/{name}', function ($name) {
        echo "<h1>Hello! $name</h1>";
    });

:doc:`Phalcon\\Mvc\\Micro <../api/Phalcon_Mvc_Micro>` предлагает набор инструментов для создания HTTP-метода (или методов),
необходимых для создания пути:

.. code-block:: php

    <?php

    // Совпадет, если HTTP-метод - GET
    $app->get('/api/products', "get_products");

    // Совпадет, если HTTP-метод - POST
    $app->post('/api/products/add', "add_product");

    // Совпадет, если HTTP-метод - PUT
    $app->put('/api/products/update/{id}', "update_product");

    // Совпадет, если HTTP-метод - DELETE
    $app->put('/api/products/remove/{id}', "delete_product");

    // Совпадет, если HTTP-метод - OPTIONS
    $app->options('/api/products/info/{id}', "info_product");

    // Совпадет, если HTTP-метод - PATCH
    $app->patch('/api/products/update/{id}', "info_product");

    // Совпадет, если HTTP-метод - GET или POST
    $app->map('/repos/store/refs',"action_product")->via(array('GET', 'POST'));


Пути с параметрами
^^^^^^^^^^^^^^^^^^
Создание параметров путей - довольно простая задача, как показывает пример выше.
Имя параметра должно находиться в скобках. Параметры также можно задавать с помощью регулярных выражений для того,
чтобы быть уверенным в наличии данных. Это показано в примере ниже:

.. code-block:: php

    <?php

    // Данный путь имеет два параметра, у каждого из которых задан формат
    $app->get('/posts/{year:[0-9]+}/{title:[a-zA-Z\-]+}', function ($year, $title) {
        echo "<h1>Title: $title</h1>";
        echo "<h2>Year: $year</h2>";
    });

Маршрут по умолчанию
^^^^^^^^^^^^^^^^^^^^
Как правило, маршрутом по умолчанию в приложении является маршрут /. Чаще всего, обращения будут 
идти именно к нему через метод GET. Этот сценарий можно описать следующим образом:

.. code-block:: php

    <?php

    // Это маршрут по умолчанию
    $app->get('/', function () {
        echo "<h1>Welcome!</h1>";
    });

Правила перезаписи (Rewrite Rules)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Следующие правила могут быть использованы вместе с Apache для перезаписи URI:

.. code-block:: apacheconf

    <IfModule mod_rewrite.c>
        RewriteEngine On
        RewriteCond %{REQUEST_FILENAME} !-f
        RewriteRule ^(.*)$ index.php?_url=/$1 [QSA,L]
    </IfModule>

Работа с заголовками ответов (Responses)
----------------------------------------
Вы можете работать с любыми заголовками ответов в обработчике: сразу сделать вывод, использовать шаблонизатор, 
подключить шаблонизатор, вернуть JSON и т.д.:

.. code-block:: php

    <?php

    // Прямой вывод
    $app->get('/say/hello', function () {
        echo "<h1>Hello! $name</h1>";
    });

    // Подключение внешнего файла
    $app->get('/show/results', function () {
        require 'views/results.php';
    });

    // Возврат JSON
    $app->get('/get/some-json', function () {
        echo json_encode(array("some", "important", "data"));
    });

В дополнение к этому, у вас есть доступ к сервису :doc:`"response" <response>`, благодаря которому вы 
можете обрабатывать ответы ещё более гибко:

.. code-block:: php

    <?php

    $app->get('/show/data', function () use ($app) {

        // Установка заголовка Content-Type
        $app->response->setContentType('text/plain')->sendHeaders();

        // Вывод содержимого файла
        readfile("data.txt");

    });

Создание перенаправлений (Redirects)
------------------------------------
Перенаправления могут быть использованы для того, чтобы перенаправить поток исполнения на другой маршрут:

.. code-block:: php

    <?php

    // Этот маршрут выполняет перенаправление на другой маршрут
    $app->post('/old/welcome', function () use ($app) {
        $app->response->redirect("new/welcome");
    });

    $app->post('/new/welcome', function () use ($app) {
        echo 'This is the new Welcome';
    });

Создание URL-адресов для маршрутов
----------------------------------
Класс :doc:`Phalcon\\Mvc\\Url <url>` может быть использован для получения URL-адреса на основе 
определенных маршрутов. Вам нужно создать имя для маршрута; опираясь на него служба "url" 
выполнить соответствующий URL:

.. code-block:: php

    <?php

    // Установка маршрута с именем "show-post"
    $app->get('/blog/{year}/{title}', function ($year, $title) use ($app) {

        //.. здесь показываем текст статьи

    })->setName('show-post');

    // Где-нибудь используем наш новый адрес
    $app->get('/', function() use ($app) {

        echo '<a href="', $app->url->get(array(
            'for' => 'show-post',
            'title' => 'php-is-a-great-framework',
            'year' => 2012
        )), '">Show the post</a>';

    });


Работа с Внедрением зависимостей (Dependency Injector)
------------------------------------------------------
В микроприложении сервисы контейнера :doc:`Phalcon\\DI\\FactoryDefault <di>` создаются неявно; 
Кроме того, вы можете создать за пределами своего приложения контейнер, который будет 
манипулировать этими сервисами:

.. code-block:: php

    <?php

    use Phalcon\DI\FactoryDefault,
        Phalcon\Mvc\Micro,
        Phalcon\Config\Adapter\Ini as IniConfig;

    $di = new FactoryDefault();

    $di->set('config', function() {
        return new IniConfig("config.ini");
    });

    $app = new Micro();

    $app->setDI($di);

    $app->get('/', function () use ($app) {
        // Читаем свойства нашего конфигурационного файла
        echo $app->config->app_name;
    });

    $app->post('/contact', function () use ($app) {
        $app->flash->success('Yes!, the contact was made!');
    });

Синтаксис массивов удобен для установки/получения сервисов из внутреннего контейнера сервисов:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Micro,
        Phalcon\Db\Adapter\Pdo\Mysql as MysqlAdapter;

    $app = new Micro();

    // Установка сервиса базы данных
    $app['db'] = function() {
        return new MysqlAdapter(array(
            "host" => "localhost",
            "username" => "root",
            "password" => "secret",
            "dbname" => "test_db"
        ));
    };

    $app->get('/blog', function () use ($app) {
        $news = $app['db']->query('SELECT * FROM news');
        foreach ($news as $new) {
            echo $new->title;
        }
    });

Обработка исключений "Не найдено"
---------------------------------
Когда пользователь пытается получить доступ к маршруту, который не определён, микроприложение 
запускает обработчик "Не найдено". Пример:

.. code-block:: php

    <?php

    $app->notFound(function () use ($app) {
        $app->response->setStatusCode(404, "Not Found")->sendHeaders();
        echo 'This is crazy, but this page was not found!';
    });

Модели в микроприложениях
-------------------------
:doc:`Модели <models>` в микроприложениях работают так же, как и в обычных. Главное - зарегистрировать автозагрузчик:

.. code-block:: php

    <?php

    $loader = new \Phalcon\Loader();

    $loader->registerDirs(array(
        __DIR__ . '/models/'
    ))->register();

    $app = new \Phalcon\Mvc\Micro();

    $app->get('/products/find', function(){

        foreach (Products::find() as $product) {
            echo $product->name, '<br>';
        }

    });

    $app->handle();

События микроприложения
-----------------------
:doc:`Phalcon\\Mvc\\Micro <../api/Phalcon_Mvc_Micro>` is able to send events to the :doc:`EventsManager <events>` (if it is present).
Events are triggered using the type "micro". The following events are supported:

+---------------------+----------------------------------------------------------------------------------------------------------------------------+----------------------+
| Event Name          | Triggered                                                                                                                  | Can stop operation?  |
+=====================+============================================================================================================================+======================+
| beforeHandleRoute   | The main method is just called, at this point the application doesn't know if there is some matched route                  | Yes                  |
+---------------------+----------------------------------------------------------------------------------------------------------------------------+----------------------+
| beforeExecuteRoute  | A route has been matched and it contains a valid handler, at this point the handler has not been executed                  | Yes                  |
+---------------------+----------------------------------------------------------------------------------------------------------------------------+----------------------+
| afterExecuteRoute   | Triggered after running the handler                                                                                        | No                   |
+---------------------+----------------------------------------------------------------------------------------------------------------------------+----------------------+
| beforeNotFound      | Triggered when any of the defined routes match the requested URI                                                           | Yes                  |
+---------------------+----------------------------------------------------------------------------------------------------------------------------+----------------------+
| afterHandleRoute    | Triggered after completing the whole process in a successful way                                                           | Yes                  |
+---------------------+----------------------------------------------------------------------------------------------------------------------------+----------------------+

In the following example, we explain how to control the application security using events:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Micro,
        Phalcon\Events\Manager as EventsManager;

    //Create a events manager
    $eventManager = new EventsManager();

    //Listen all the application events
    $eventManager->attach('micro', function($event, $app) {

        if ($event->getType() == 'beforeExecuteRoute') {
            if ($app->session->get('auth') == false) {

                $app->flashSession->error("The user isn't authenticated");
                $app->response->redirect("/");

                //Return (false) stop the operation
                return false;
            }
        }

    });

    $app = new Micro();

    //Bind the events manager to the app
    $app->setEventsManager($eventsManager);

Middleware events
-----------------
In addition to the events manager, events can be added using the methods 'before', 'after' and 'finish':

.. code-block:: php

    <?php

    $app = new Phalcon\Mvc\Micro();

    //Executed before every route executed
    //Return false cancels the route execution
    $app->before(function() use ($app) {
        if ($app['session']->get('auth') == false) {
            return false;
        }
        return true;
    });

    $app->map('/api/robots', function(){
        return array(
            'status' => 'OK'
        );
    });

    $app->after(function() use ($app) {
        //This is executed after the route is executed
        echo json_encode($app->getReturnedValue());
    });

    $app->finish(function() use ($app) {
        //This is executed when the request has been served
    });

You can call the methods several times to add more events of the same type.

Code for middlewares can be reused using separate classes:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Micro\MiddlewareInterface;

    /**
     * CacheMiddleware
     *
     * Caches pages to reduce processing
     */
    class CacheMiddleware implements MiddlewareInterface
    {
        public function call($application)
        {

            $cache = $application['cache'];
            $router = $application['router'];

            $key = preg_replace('/^[a-zA-Z0-9]/', '', $router->getRewriteUri());

            //Check if the request is cached
            if ($cache->exists($key)) {
                echo $cache->get($key);
                return false;
            }

            return true;
        }
    }

Then add the instance to the application:

.. code-block:: php

    <?php

    $app->before(new CacheMiddleware());

The following middleware events are available:

+---------------------+----------------------------------------------------------------------------------------------------------------------------+----------------------+
| Event Name          | Triggered                                                                                                                  | Can stop operation?  |
+=====================+============================================================================================================================+======================+
| before              | Before executing the handler. It can be used to control the access to the application                                      | Yes                  |
+---------------------+----------------------------------------------------------------------------------------------------------------------------+----------------------+
| after               | Executed after the handler is executed. It can be used to prepare the response                                             | No                   |
+---------------------+----------------------------------------------------------------------------------------------------------------------------+----------------------+
| finish              | Executed after sending the response. It can be used to perform clean-up                                                    | No                   |
+---------------------+----------------------------------------------------------------------------------------------------------------------------+----------------------+

Using Controllers as Handlers
-----------------------------
Medium applications using the Micro\\MVC approach may require organize handlers in controllers.
You can use :doc:`Phalcon\\Mvc\\Micro\\Collection <../api/Phalcon_Mvc_Micro_Collection>` to group handlers that belongs to controllers:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Micro\Collection as MicroCollection;

    $posts = new MicroCollection();

    //Set the main handler. ie. a controller instance
    $posts->setHandler(new PostsController());

    //Set a common prefix for all routes
    $posts->setPrefix('/posts');

    //Use the method 'index' in PostsController
    $posts->get('/', 'index');

    //Use the method 'show' in PostsController
    $posts->get('/show/{slug}', 'show');

    $app->mount($posts);

The controller 'PostsController' might look like this:

.. code-block:: php

    <?php

    class PostsController extends Phalcon\Mvc\Controller
    {

        public function index()
        {
            //...
        }

        public function show($slug)
        {
            //...
        }
    }

The example driver directly instantiated, Collection also is provided in the ability to load
the drivers only if the route is matched:

.. code-block:: php

    <?php

    $posts->setHandler('PostsController', true);
    $posts->setHandler('Blog\Controllers\PostsController', true);

Returning Responses
-------------------
Handlers may return raw responses using :doc:`Phalcon\\Http\\Response <response>` or a component that implements the relevant interface:

.. code-block:: php

    <?php

    use Phalcon\Mvc\Micro,
        Phalcon\Http\Response;

    $app = new Micro();

    //Return a response
    $app->get('/welcome/index', function() {

        $response = new Response();

        $response->setStatusCode(401, "Unauthorized");

        $response->setContent("Access is not authorized");

        return $response;
    });

Rendering Views
---------------
:doc:`Phalcon\\Mvc\\View <views>` can be used to render views, the following example shows how to do that:

.. code-block:: php

    <?php

    $app = new Phalcon\Mvc\Micro();

    $app['view'] = function() {
        $view = new \Phalcon\Mvc\View();
        $view->setViewsDir('app/views/');
        return $view;
    };

    //Return a rendered view
    $app->get('/products/show', function() use ($app) {

        // Render app/views/products/show.phtml passing some variables
        echo $app['view']->getRender('products', 'show', array(
            'id' => 100,
            'name' => 'Artichoke'
        ));

    });

Внешние источники
-----------------
* :doc:`Creating a Simple REST API <tutorial-rest>` is a tutorial that explains how to create a micro application to implement a RESTful web service.
* `Stickers Store <http://store.phalconphp.com>`_ is a very simple micro-application making use of the micro-mvc approach [`Github <https://github.com/phalcon/store>`_].
