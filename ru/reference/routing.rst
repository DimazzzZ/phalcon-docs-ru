Роутинг (Routing, Маршрутизация)
================================
Компонент маршрутизации позволяет определять маршруты, которые будут привязаны к контроллерам, или обработчикам для получения
запроса. Маршрутизатор просто разбирает URI для определения информации. Маршрутизатор имеет два режима: MVC
режим и режим совпадения. Первый режим идеально подходит для работы с MVC приложениями.

Определение маршрутов
---------------------
:doc:`Phalcon\\Mvc\\Router <../api/Phalcon_Mvc_Router>` предоставляет расширенные возможности маршрутизации. В MVC режиме вы
можете определить маршруты и направить их на контроллеры/действия, которые вам требуются. Маршруты определяются следующим образом:

.. code-block:: php

    <?php

    // Создание маршрутизатора
    $router = new \Phalcon\Mvc\Router();

    // Определение правила маршрутизации
    $router->add(
        "/admin/users/my-profile",
        array(
            "controller" => "users",
            "action"     => "profile",
        )
    );

    // Еще одно правило
    $router->add(
        "/admin/users/change-password",
        array(
            "controller" => "users",
            "action"     => "changePassword",
        )
    );

    $router->handle();

Метод add() принимает в качестве первого параметра шаблон ссылки, вторым параметром настройки этого маршрута.
В этом случае, если URI соответствует /admin/users/my-profile, и будет выполнен контроллер "users", а в нём действие "profile".
Маршрутизатор не выполняет действие контроллера, он только собирает эту информацию, чтобы сообщить правильные параметры в компонент
:doc:`Phalcon\\Mvc\\Dispatcher <../api/Phalcon_Mvc_Dispatcher>`.

Приложение может иметь множество маршрутов, определения их по одному может быть достаточно трудоемкой задачей. В таких случаях мы можем
создавать более гибкие маршруты:

.. code-block:: php

    <?php

    // Создание маршрутизатора
    $router = new \Phalcon\Mvc\Router();

    // Определение правила маршрутизации
    $router->add(
        "/admin/:controller/a/:action/:params",
        array(
            "controller" => 1,
            "action"     => 2,
            "params"     => 3,
        )
    );

In the example above, using wildcards we make a route valid for many URIs. For example, by accessing the
following URL (/admin/users/a/delete/dave/301) then:

В примере, приведенном выше, с помощью подстановочных элементов мы делаем маршрут подходящим для множества ссылок. Например, при получении
URL (/admin/users/a/delete/dave/301), маршрутизатор раберёт его в:

+-------------------------+----------+
| Контроллер / Controller | users    |
+-------------------------+----------+
| Действие / Action       | delete   |
+-------------------------+----------+
| Параметр / Parameter    | dave     |
+-------------------------+----------+
| Параметр / Parameter    | 301      |
+-------------------------+----------+

Метод add() принимает шаблон, который по желанию может быть написан с использованием регулярных выражений. Все
маршруты должны начинаться с косой черты (/). Регулярные выражения должны соответствовать формату  `регулярных выражений PCRE`_.
Отметим, что это не нужно добавлять в регулярные выражения разделители. Все маршруты не зависят от регистра.

Второй параметр определяет, какие из подходящих частей следует "привязать" к controller/action/parameters. Соответствующие
части берутся из "заполнителей" или по маскам ограничивающимися круглыми скобками. В примере, приведенном выше, 
первой части соответствует контроллеру (:controller), второй действию и так далее.

Заполнители помогают написанию регулярных выражений, они более читабельны для разработчиков и проще
для понимания. Существуют такие заполнители:

+--------------+---------------------+--------------------------------------------------------------------+
| Placeholder  | Regular Expression  | Usage                                                              |
+==============+=====================+====================================================================+
| /:module     | /([a-zA-Z0-9\_\-]+) | Matches a valid module name with alpha-numeric characters only     |
+--------------+---------------------+--------------------------------------------------------------------+
| /:controller | /([a-zA-Z0-9\_\-]+) | Matches a valid controller name with alpha-numeric characters only |
+--------------+---------------------+--------------------------------------------------------------------+
| /:action     | /([a-zA-Z0-9\_]+)   | Matches a valid action name with alpha-numeric characters only     |
+--------------+---------------------+--------------------------------------------------------------------+
| /:params     | (/.*)*              | Matches a list of optional words separated by slashes              |
+--------------+---------------------+--------------------------------------------------------------------+
| /:namespace  | /([a-zA-Z0-9\_\-]+) | Matches a single level namespace name                              |
+--------------+---------------------+--------------------------------------------------------------------+
| /:int        | /([0-9]+)           | Matches an integer parameter                                       |
+--------------+---------------------+--------------------------------------------------------------------+

Названия контроллеров "camelized", это означает, что символы (-) и (_) удаляются, и следующий после них символ
преобразуется в верхний регистр. Например, some_controller преобразуется в SomeController.

Поскольку вы можете использовать множество маршрутов, добавляя их методом add(), порядок, в котором маршруты добавляются указывает
их актуальность, последние добавленные маршруты имеют больший приоритет, чем добавленные ранее. В Внутри все определенные маршруты
перемещаются в обратном порядке, пока :doc:`Phalcon\\Mvc\\Router <../api/Phalcon_Mvc_Router>` не найдёт
тот, который соответствует данному URI и использует его, игнорируя остальные.

Именованные параметры
^^^^^^^^^^^^^^^^^^^^^
В примере ниже показано, как определить имена для параметров маршрутов:

.. code-block:: php

    <?php

    $router->add(
        "/news/([0-9]{4})/([0-9]{2})/([0-9]{2})/:params",
        array(
        	"controller" => "posts",
        	"action"     => "show",
        	"year"       => 1, // ([0-9]{4})
        	"month"      => 2, // ([0-9]{2})
        	"day"        => 3, // ([0-9]{2})
        	"params"     => 4, // :params
        )
    );

В приведенном выше примере, в маршруте не определены части для "контроллера" или "действия". Эти параметры заменяются
фиксированными значениями ("posts" и "show"). Пользователь не будет видеть вызванный контроллер.
Внутри контроллера именованные параметры можно получить следующим образом:

.. code-block:: php

    <?php

    class PostsController extends \Phalcon\Mvc\Controller
    {

        public function indexAction()
        {

        }

        public function showAction()
        {

            // Возвращает параметр "year"
            $year = $this->dispatcher->getParam("year");

            // Возвращает параметр "month"
            $month = $this->dispatcher->getParam("month");

            // Возвращает параметр "day"
            $day = $this->dispatcher->getParam("day");

        }

    }

Обратите внимание, что значения параметров получаются из диспетчера. Это происходит потому, что это
компонент, который, непосредственно запускает в работу ваше приложение.
Кроме того, существует и другой способ создавать именованные параметры, например, как часть правила маршрутизации:

.. code-block:: php

    <?php

    $router->add(
        "/documentation/{chapter}/{name}.{type:[a-z]+}",
        array(
            "controller" => "documentation",
            "action"     => "show"
        )
    );

Вы можете получить доступ к их значения так же, как раньше:

.. code-block:: php

    <?php

    class DocumentationController extends \Phalcon\Mvc\Controller
    {

        public function showAction()
        {

            // Returns "name" parameter
            $year = $this->dispatcher->getParam("name");

            // Returns "type" parameter
            $year = $this->dispatcher->getParam("type");

        }

    }

Краткий сантаксис
^^^^^^^^^^^^^^^^^
Если вам не нравится использование массивов для определения правил маршрута, альтернативный синтаксис также доступен.
Следующие примеры дают одинаковый результат:

.. code-block:: php

    <?php

    // Краткий синтаксис
    $router->add("/posts/{year:[0-9]+}/{title:[a-z\-]+}", "Posts::show");

    // Использование массива
    $router->add(
        "/posts/([0-9]+)/([a-z\-]+)",
        array(
           "controller" => "posts",
           "action"     => "show",
           "year"       => 1,
           "title"      => 2,
        )
    );

Совмещение массивов и краткого сантаксиса
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Массив и краткий синтаксис может быть смешанны, в данном случае, обратите внимание, что именованные параметры автоматически
добавляются в маршрут в соответствии с положением, на котором они были определены:

.. code-block:: php

    <?php

    // В качестве первой позиции выступает параметр 'country'
    $router->add('/news/{country:[a-z]{2}}/([a-z+])/([a-z\-+])',
        array(
            'section' => 2, // Это уже позиция номер 2
            'article' => 3
        )
    );

Маршрутизация модулей
^^^^^^^^^^^^^^^^^^^^^
Вы можете определить маршруты, пути которых включают в себя модули. Это особенно подходит для мульти-модульных приложений.
Возможно так же определить маршрут по умолчанию, который включает в себя модуль шаблона:

.. code-block:: php

    <?php

    $router = new Phalcon\Mvc\Router(false);

    $router->add('/:module/:controller/:action/:params', array(
        'module' => 1,
        'controller' => 2,
        'action' => 3,
        'params' => 4
    ));

В этом случае маршрут всегда должен иметь имя модуля в качестве части URL-адреса. Например, в следующем
URL: /admin/users/edit/sonny, будут обработан как:

+------------+---------------+
| Module     | admin         |
+------------+---------------+
| Controller | users         |
+------------+---------------+
| Action     | edit          |
+------------+---------------+
| Parameter  | sonny         |
+------------+---------------+

Или вы можете привязать конкретные маршруты к конкретным модулям:

.. code-block:: php

    <?php

    $router->add("/login", array(
        'module' => 'backend',
        'controller' => 'login',
        'action' => 'index',
    ));

    $router->add("/products/:action", array(
        'module' => 'frontend',
        'controller' => 'products',
        'action' => 1,
    ));

Или привязать к конкретному пространству имен:

.. code-block:: php

    <?php

    $router->add("/:namespace/login", array(
        'namespace' => 1,
        'controller' => 'login',
        'action' => 'index'
    ));

Пространства имён и названия классов должны передаваться раздельно:

.. code-block:: php

    <?php

    $router->add("/login", array(
        'namespace' => 'Backend\Controllers',
        'controller' => 'login',
        'action' => 'index'
    ));

Разделение по HTTP методам
^^^^^^^^^^^^^^^^^^^^^^^^^^
При добавлении маршрута, используя метод add(), маршрут будет активен для любого HTTP-метода. Иногда можно использовать маршрут для
конкретного метода, это особенно полезно при создании RESTful приложений:

.. code-block:: php

    <?php

    // Маршрут соответствует только HTTP методу GET
    $router->addGet("/products/edit/{id}", "Posts::edit");

    // Маршрут соответствует только HTTP методу POST
    $router->addPost("/products/save", "Posts::save");

    // Маршрут соответствует сразу двум HTTP методам POST и PUT
    $router->add("/products/update")->via(array("POST", "PUT"));

Использование преобразований
^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Метод convert позволяет трансформировать параметры маршрута до передачи их диспетчеру, следующие пример показывает вариант использования:

.. code-block:: php

    <?php

    // Название действия разрешает использование "-": /products/new-ipod-nano-4-generation
    $router
        ->add('/products/{slug:[a-z\-]+}', array(
            'controller' => 'products',
            'action' => 'show'
        ))
        ->convert('slug', function($slug) {
            // Удаляем тире из выбранного параметра
            return str_replace('-', '', $slug);
        });

Группы маршрутов
^^^^^^^^^^^^^^^^
Если набор маршрутов имеют общие пути они могут быть сгруппированы для легкой поддержки:

.. code-block:: php

    <?php

    $router = new \Phalcon\Mvc\Router();

    // Создаётся группа с общим модулем и контроллером
    $blog = new \Phalcon\Mvc\Router\Group(array(
        'module' => 'blog',
        'controller' => 'index'
    ));

    // Маршруты начинаются с /blog
    $blog->setPrefix('/blog');

    // Добавление маршрута в группу
    $blog->add('/save', array(
        'action' => 'save'
    ));

    // Еще один маршрут
    $blog->add('/edit/{id}', array(
        'action' => 'edit'
    ));

    // Маршрут для действия по умолчанию
    $blog->add('/blog', array(
        'controller' => 'about',
        'action' => 'index'
    ));

    // Добавление группы в общие правила маршрутизации
    $router->mount($blog);

Соответствие маршрутов
----------------------
Текущий URI должен передаётся маршрутизатору для сопоставления его маршруту.
По умолчанию, URI для обработки берется из переменной $_GET['_url'], полученной с использование mod_rewrite.
Для Phalcon подходят очень простые правила mod_rewrite:

.. code-block:: apacheconf

    RewriteEngine On
    RewriteCond   %{REQUEST_FILENAME} !-d
    RewriteCond   %{REQUEST_FILENAME} !-f
    RewriteRule   ^(.*)$ index.php?_url=/$1 [QSA,L]

В следующем примере показано, как использовать этот компонент автономно:

.. code-block:: php

    <?php

    // Создание маршрутизатора
    $router = new \Phalcon\Mvc\Router();

    // Тут устанавливаются правила маршрутизации
    // ...

    // Бдует использован $_GET["_url"]
    $router->handle();

    // Можно указать параметр самостоятельно
    $router->handle("/employees/edit/17");

    // Получаем выбранный контроллер
    echo $router->getControllerName();

    // .. и соответсвющее действие
    echo $router->getActionName();

    // Получаем сам выбранный для ссылки маршрут
    $route = $router->getMatchedRoute();

Именованные маршруты
--------------------
Each route that is added to the router is stored internally as an object :doc:`Phalcon\\Mvc\\Router\\Route <../api/Phalcon_Mvc_Router_Route>`.
That class encapsulates all the details of each route. For instance, we can give a name to a path to identify it uniquely in our application.
This is especially useful if you want to create URLs from it.

.. code-block:: php

    <?php

    $route = $router->add("/posts/{year}/{title}", "Posts::show");

    $route->setName("show-posts");

    //or just

    $router->add("/posts/{year}/{title}", "Posts::show")->setName("show-posts");

Then, using for example the component :doc:`Phalcon\\Mvc\\Url <../api/Phalcon_Mvc_Url>` we can build routes from its name:

.. code-block:: php

    <?php

    // returns /posts/2012/phalcon-1-0-released
    echo $url->get(array(
        "for" => "show-posts",
        "year" => "2012", "title" =>
        "phalcon-1-0-released"
    ));

Примеры использования
---------------------
Ниже приведены примеры пользовательских маршрутов:

.. code-block:: php

    <?php

    // пример - "/system/admin/a/edit/7001"
    $router->add(
        "/system/:controller/a/:action/:params",
        array(
            "controller" => 1,
            "action"     => 2,
            "params"     => 3
        )
    );

    // пример - "/es/news"
    $router->add(
        "/([a-z]{2})/:controller",
        array(
            "controller" => 2,
            "action"     => "index",
            "language"   => 1
        )
    );

    // пример - "/es/news"
    $router->add(
        "/{language:[a-z]{2}}/:controller",
        array(
            "controller" => 2,
            "action"     => "index"
        )
    );

    // пример - "/admin/posts/edit/100"
    $router->add(
        "/admin/:controller/:action/:int",
        array(
            "controller" => 1,
            "action"     => 2,
            "id"         => 3
        )
    );

    // пример - "/posts/2010/02/some-cool-content"
    $router->add(
        "/posts/([0-9]{4})/([0-9]{2})/([a-z\-]+)",
        array(
            "controller" => "posts",
            "action"     => "show",
            "year"       => 1,
            "month"      => 2,
            "title"      => 4
        )
    );

    // пример - "/manual/en/translate.adapter.html"
    $router->add(
        "/manual/([a-z]{2})/([a-z\.]+)\.html",
        array(
            "controller" => "manual",
            "action"     => "show",
            "language"   => 1,
            "file"       => 2
        )
    );

    // пример - /feed/fr/le-robots-hot-news.atom
    $router->add(
        "/feed/{lang:[a-z]+}/{blog:[a-z\-]+}\.{type:[a-z\-]+}",
        "Feed::get"
    );

    // пример - /api/v1/users/peter.json
    $router->add('/api/(v1|v2)/{method:[a-z]+}/{param:[a-z]+}\.(json|xml)', array(
        'controller' => 'api',
        'version' => 1,
        'format' => 4
    ));

.. highlights::
    Beware of characters allowed in regular expression for controllers and namespaces. As these
    become class names and in turn they're passed through the file system could be used by attackers to
    read unauthorized files. A safe regular expression is: /([a-zA-Z0-9\_\-]+)

Поведение по умолчанию
----------------------
:doc:`Phalcon\\Mvc\\Router <../api/Phalcon_Mvc_Router>` has a default behavior providing a very simple routing that
always expects a URI that matches the following pattern: /:controller/:action/:params

For example, for a URL like this *http://phalconphp.com/documentation/show/about.html*, this router will translate it as follows:

+------------+---------------+
| Controller | documentation |
+------------+---------------+
| Action     | show          |
+------------+---------------+
| Parameter  | about.html    |
+------------+---------------+

If you don't want use this routes as default in your application, you must create the router passing false as parameter:

.. code-block:: php

    <?php

    // Create the router without default routes
    $router = new \Phalcon\Mvc\Router(false);

Указание маршрута по умолчанию
------------------------------
When your application is accessed without any route, the '/' route is used to determine what paths must be used to show the initial page
in your website/application:

.. code-block:: php

    <?php

    $router->add("/", array(
        'controller' => 'index',
        'action' => 'index'
    ));

404 страница
------------
If none of the routes specified in the router are matched, you can define a group of paths to be used in this scenario:

.. code-block:: php

    <?php

    //Set 404 paths
    $router->notFound(array(
        "controller" => "index",
        "action" => "route404"
    ));

Установка параметров по умолчанию
---------------------------------
It's possible to define default values for common paths like module, controller or action. When a route is missing any of
those paths they can be automatically filled by the router:

.. code-block:: php

    <?php

    //Individually
    $router->setDefaultModule("backend");
    $router->setDefaultNamespace('Backend\Controllers');
    $router->setDefaultController("index");
    $router->setDefaultAction("index");

    //Using an array
    $router->setDefaults(array(
        "controller" => "index",
        "action" => "index"
    ));

Использование конечного /
-------------------------
Sometimes a route could be accessed with extra/trailing slashes and the end of the route, those extra slashes would lead to produce
a not-found status in the dispatcher. You can set up the router to automatically remove the slashes from the end of handled route:

.. code-block:: php

    <?php

    $router = new \Phalcon\Mvc\Router();

    //Remove trailing slashes automatically
    $router->removeExtraSlashes(true);

Or, you can modify specific routes to optionally accept trailing slashes:

.. code-block:: php

    <?php

    $router->add(
        "/{language:[a-z]{2}}/:controller[/]{0,1}",
        array(
            "controller" => 2,
            "action"     => "index"
        )
    );

Источники URI
-------------
By default the URI information is obtained from the $_GET['_url'] variable, this is passed by the Rewrite-Engine to 
Phalcon, you can also use $_SERVER['REQUEST_URI'] if required:

.. code-block:: php

    <?php

    $router->setUriSource(Router::URI_SOURCE_GET_URL); // use $_GET['_url'] (default)
    $router->setUriSource(Router::URI_SOURCE_SERVER_REQUEST_URI); // use $_SERVER['REQUEST_URI'] (default)

Or you can manually pass a URI to the 'handle' method:

.. code-block:: php

    <?php

    $router->handle('/some/route/to/handle');

Тестирование маршрутов
----------------------
Since this component has no dependencies, you can create a file as shown below to test your routes:

.. code-block:: php

    <?php

    //These routes simulate real URIs
    $testRoutes = array(
        '/',
        '/index',
        '/index/index',
        '/index/test',
        '/products',
        '/products/index/',
        '/products/show/101',
    );

    $router = new Phalcon\Mvc\Router();

    //Add here your custom routes
    //...

    //Testing each route
    foreach ($testRoutes as $testRoute) {

        //Handle the route
        $router->handle($testRoute);

        echo 'Testing ', $testRoute, '<br>';

        //Check if some route was matched
        if ($router->wasMatched()) {
            echo 'Controller: ', $router->getControllerName(), '<br>';
            echo 'Action: ', $router->getActionName(), '<br>';
        } else {
            echo 'The route wasn\'t matched by any route<br>';
        }
        echo '<br>';

    }

Маршруты на аннотациях
----------------------
This component provides a variant that's integrated with the :doc:`annotations <annotations>` service. Using this strategy
you can write the routes directly in the controllers instead of adding them in the service registration:

.. code-block:: php

    <?php

    $di['router'] = function() {

        //Use the annotations router
        $router = new \Phalcon\Mvc\Router\Annotations(false);

        //Read the annotations from ProductsController if the uri starts with /api/products
        $router->addResource('Products', '/api/products');

        return $router;
    };

The annotations can be defined in the following way:

.. code-block:: php

    <?php

    /**
     * @RoutePrefix("/api/products")
     */
    class ProductsController
    {

        /**
         * @Get("/")
         */
        public function indexAction()
        {

        }

        /**
         * @Get("/edit/{id:[0-9]+}", name="edit-robot")
         */
        public function editAction($id)
        {

        }

        /**
         * @Route("/save", methods={"POST", "PUT"}, name="save-robot")
         */
        public function saveAction()
        {

        }

        /**
         * @Route("/delete/{id:[0-9]+}", methods="DELETE",
         *      conversors={id="MyConversors::checkId"})
         */
        public function deleteAction($id)
        {

        }

        public function infoAction($id)
        {

        }

    }

Only methods marked with valid annotations are used as routes. List of annotations supported:

+--------------+---------------------------------------------------------------------------------------------------+--------------------------------------------------------------------+
| Name         | Description                                                                                       | Usage                                                              |
+==============+===================================================================================================+====================================================================+
| RoutePrefix  | A prefix to be prepended to each route uri. This annotation must be placed at the class' docblock | @RoutePrefix("/api/products")                                      |
+--------------+---------------------------------------------------------------------------------------------------+--------------------------------------------------------------------+
| Route        | This annotation marks a method as a route. This annotation must be placed in a method docblock    | @Route("/api/products/show")                                       |
+--------------+---------------------------------------------------------------------------------------------------+--------------------------------------------------------------------+
| Get          | This annotation marks a method as a route restricting the HTTP method to GET                      | @Get("/api/products/search")                                       |
+--------------+---------------------------------------------------------------------------------------------------+--------------------------------------------------------------------+
| Post         | This annotation marks a method as a route restricting the HTTP method to POST                     | @Post("/api/products/save")                                        |
+--------------+---------------------------------------------------------------------------------------------------+--------------------------------------------------------------------+
| Put          | This annotation marks a method as a route restricting the HTTP method to PUT                      | @Put("/api/products/save")                                         |
+--------------+---------------------------------------------------------------------------------------------------+--------------------------------------------------------------------+
| Delete       | This annotation marks a method as a route restricting the HTTP method to DELETE                   | @Delete("/api/products/delete/{id}")                               |
+--------------+---------------------------------------------------------------------------------------------------+--------------------------------------------------------------------+
| Options      | This annotation marks a method as a route restricting the HTTP method to OPTIONS                  | @Option("/api/products/info")                                      |
+--------------+---------------------------------------------------------------------------------------------------+--------------------------------------------------------------------+

For annotations that add routes, the following parameters are supported:

+--------------+---------------------------------------------------------------------------------------------------+--------------------------------------------------------------------+
| Name         | Description                                                                                       | Usage                                                              |
+==============+===================================================================================================+====================================================================+
| methods      | Define one or more HTTP method that route must meet with                                          | @Route("/api/products", methods={"GET", "POST"})                   |
+--------------+---------------------------------------------------------------------------------------------------+--------------------------------------------------------------------+
| name         | Define a name for the route                                                                       | @Route("/api/products", name="get-products")                       |
+--------------+---------------------------------------------------------------------------------------------------+--------------------------------------------------------------------+
| paths        | An array of paths like the one passed to Phalcon\\Mvc\\Router::add                                | @Route("/posts/{id}/{slug}", paths={module="backend"})             |
+--------------+---------------------------------------------------------------------------------------------------+--------------------------------------------------------------------+
| conversors   | A hash of conversors to be applied to the parameters                                              | @Route("/posts/{id}/{slug}", conversors={id="MyConversor::getId"}) |
+--------------+---------------------------------------------------------------------------------------------------+--------------------------------------------------------------------+

If routes map to controllers in modules is better use the addModuleResource method:

.. code-block:: php

    <?php

    $di['router'] = function() {

        //Use the annotations router
        $router = new \Phalcon\Mvc\Router\Annotations(false);

        //Read the annotations from Backend\Controllers\ProductsController if the uri starts with /api/products
        $router->addModuleResource('backend', 'Products', '/api/products');

        return $router;
    };

Создание собственного маршрутизатора
------------------------------------
Для создания адаптера необходимо реализовать интерфейс :doc:`Phalcon\\Mvc\\RouterInterface <../api/Phalcon_Mvc_RouterInterface>`. Созданным классом
надо подменить маршрутизатор ('router') в момент инициализации приложения.

.. _регулярных выражений PCRE: http://www.php.net/manual/en/book.pcre.php
