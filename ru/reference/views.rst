Использование представлений (Views)
===================================
Представления (views) — это пользовательский интерфейс вашего приложения. Чаще всего это HTML-файлы со вставками PHP-кода, который выполняет только задачи, связанные с выводом данных. Представления управляют работой по передаче данных в браузер или другой инструмент, использующийся для выполнения запросов к вашему приложению.

The :doc:`Phalcon\\Mvc\\View <../api/Phalcon_Mvc_View>` отвечает за управление слоем представления в вашем MVC-приложении.

Компонент поддерживает иерархическую структуру файлов. Эта иерархия определяет местоположение как общих шаблонов, так и шаблонов контроллеров, расположенных в папках с соответствующими названиями. 

Совместное использование Представлений и Контроллеров
-----------------------------------------------------
Как только какой-то определённый контроллер завершает свой цикл работы, Phalcon автоматически передаёт управление компоненту представления. Этот компонент ищет в папке представлений такую, название которой совпадает последним исполнявшимся контроллером, а затем — файл, имя которого соответствует последнему выполненному действию (action). Например, запрос по URL *http://127.0.0.1/blog/posts/show/301* Phalcon будет разбирать следующим образом:

+-------------------+-----------+
| Адрес сервера     | 127.0.0.1 |
+-------------------+-----------+
| Папка Phalcon     | blog      |
+-------------------+-----------+
| Контроллер        | posts     |
+-------------------+-----------+
| Действие          | show      |
+-------------------+-----------+
| Параметр          | 301       |
+-------------------+-----------+

Dispatcher будет искать "PostsController" и его действие "showAction". Листинг простейшего контроллера для этого примера:

.. code-block:: php

    <?php

    class PostsController extends \Phalcon\Mvc\Controller
    {

        public function indexAction()
        {

        }

        public function showAction($postId)
        {
            // Передать параметр $postId в представление
            $this->view->setVar("postId", $postId);
        }

    }

Метод setVar позволяет создавать переменные, необходимые для представления, которые могут быть использованы в шаблоне. Это и продемонстрировано выше, на примере передачи в шаблон параметра $postId.


В качестве движка для шаблона :doc:`Phalcon\\Mvc\\View <../api/Phalcon_Mvc_View>` использует сам PHP, поэтому представления должны иметь расширение .phtml.
Если в качестве папки с представлениями используется *app/views*, то компонент автоматически будет искать следующие 3 файла.

+-------------------+-------------------------------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Название          | Файл                          | Описание                                                                                                                                                                             |
+===================+===============================+======================================================================================================================================================================================+
| Action View       | app/views/posts/show.phtml    | Представление, связанное с конкретным действием контроллера. Используется только при выполнении этого действия.                                                                      |
+-------------------+-------------------------------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Controller Layout | app/views/layouts/posts.phtml | Представление, связанное с контроллером. Используется для любого действия контроллера "posts". Код, реализованный в layout будет повторно использован для всех действий контроллера. |
+-------------------+-------------------------------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Main Layout       | app/views/index.phtml         | Основной шаблон. Используется для любого контроллера или действия в приложении.                                                                                                      |
+-------------------+-------------------------------+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+


Вам совершенно не обязательно использовать все упомянутые выше файлы. :doc:`Phalcon\\Mvc\\View <../api/Phalcon_Mvc_View>` просто перейдёт на следующий уровень представлений в иерархии файлов.
В том случае, если существуют все три файла представлений, то они будут обработаны следующим образом:

.. code-block:: html+php

    <!-- app/views/posts/show.phtml -->

    <h3>This is show view!</h3>

    <p>I have received the parameter <?php echo $postId ?></p>

.. code-block:: html+php

    <!-- app/views/layouts/posts.phtml -->

    <h2>This is the "posts" controller layout!</h2>

    <?php echo $this->getContent() ?>

.. code-block:: html+php

    <!-- app/views/index.phtml -->
    <html>
        <head>
            <title>Example</title>
        </head>
        <body>

            <h1>This is main layout!</h1>

            <?php echo $this->getContent() ?>

        </body>
    </html>

Обратите внимание на строчки, в которых происходит вызов метода *$this->getContent()*. Он указывает :doc:`Phalcon\\Mvc\\View <../api/Phalcon_Mvc_View>`
где необходимо вставить содержимое представления, которое исполнялось выше по иерархии. Вывод для нашего примера будет представлять собой следующее:

.. figure:: ../_static/img/views-1.png
   :align: center

Сгенерированный HTML-код по этому запросу:

.. code-block:: html+php

    <!-- app/views/index.phtml -->
    <html>
        <head>
            <title>Example</title>
        </head>
        <body>

            <h1>This is main layout!</h1>

            <!-- app/views/layouts/posts.phtml -->

            <h2>This is the "posts" controller layout!</h2>

            <!-- app/views/posts/show.phtml -->

            <h3>This is show view!</h3>

            <p>I have received the parameter 101</p>

        </body>
    </html>

Использование Шаблонов
----------------------
Шаблоны — это представления, которые могут быть использованы для предоставления общего доступа к коду представлений. Они выступают в роли layouts для контроллеров, поэтому вам необходимо помещать их в папку для layouts.

.. code-block:: php

    <?php

    class PostsController extends \Phalcon\Mvc\Controller
    {
        public function initialize()
        {
            $this->view->setTemplateAfter('common');
        }

        public function lastAction()
        {
            $this->flash->notice("These are the latest posts");
        }
    }

.. code-block:: html+php

    <!-- app/views/index.phtml -->
    <!DOCTYPE html>
    <html>
        <head>
            <title>Blog's title</title>
        </head>
        <body>
            <?php echo $this->getContent() ?>
        </body>
    </html>

.. code-block:: html+php

    <!-- app/views/layouts/common.phtml -->

    <ul class="menu">
        <li><a href="/">Home</a></li>
        <li><a href="/articles">Articles</a></li>
        <li><a href="/contact">Contact us</a></li>
    </ul>

    <div class="content"><?php echo $this->getContent() ?></div>

.. code-block:: html+php

    <!-- app/views/layouts/posts.phtml -->

    <h1>Blog Title</h1>

    <?php echo $this->getContent() ?>

.. code-block:: html+php

    <!-- app/views/layouts/posts/last.phtml -->

    <article>
        <h2>This is a title</h2>
        <p>This is the post content</p>
    </article>

    <article>
        <h2>This is another title</h2>
        <p>This is another post content</p>
    </article>

На выходе получится следующее:

.. code-block:: html+php

    <!-- app/views/index.phtml -->
    <!DOCTYPE html>
    <html>
        <head>
            <title>Blog's title</title>
        </head>
        <body>

            <!-- app/views/layouts/common.phtml -->

            <ul class="menu">
                <li><a href="/">Home</a></li>
                <li><a href="/articles">Articles</a></li>
                <li><a href="/contact">Contact us</a></li>
            </ul>

            <div class="content">

                <!-- app/views/layouts/posts.phtml -->

                <h1>Blog Title</h1>

                <!-- app/views/layouts/posts/last.phtml -->

                <article>
                    <h2>This is a title</h2>
                    <p>This is the post content</p>
                </article>

                <article>
                    <h2>This is another title</h2>
                    <p>This is another post content</p>
                </article>

            </div>

        </body>
    </html>

Использование частичных шаблонов (Partials)
-------------------------------------------
Частичные шаблоны (Partial templates) — это ещё один способ дробления процесса отрисовки на простые и более управляемые части, которые впоследствии могут быть использованы в различных частях приложения. С помощью partial вы можете переместить код отрисовки какой-то конкретной части в отдельный, отвечающий за это, файл.

Один из способов использования partials — это отнестись к ним, как к некоторому подобию подпрограммы. Иными словами — вынести детали реализации из представления, с целью сделать код более простым для понимания. Например, вы могли бы получить представление, выглядещее следующим образом:

.. code-block:: html+php

    <?php $this->partial("shared/ad_banner") ?>

    <h1>Robots</h1>

    <p>Check out our specials for robots:</p>
    ...

    <?php $this->partial("shared/footer") ?>

Передача значений переменных из контроллера в представление
-----------------------------------------------------------
:doc:`Phalcon\\Mvc\\View <../api/Phalcon_Mvc_View>` позволяет использовать в каждом контроллере переменную представления  ($this->view). Вы можете использовать этот объект, чтобы устанавливать значения переменных для представления непосредственно из действия контроллера, используя метод setVar().

.. code-block:: php

    <?php

    class PostsController extends \Phalcon\Mvc\Controller
    {

        public function indexAction()
        {

        }

        public function showAction()
        {
            // Передать все посты в представление
            $this->view->setVar("posts", Posts::find());
        }

    }

Переменная, название которой 
Первым параметром метода setVar() передаётся название переменной, которая будет создана и может быть использована в представлении. Эта переменная может быть любого типа: от простых строк или целых чисел до более сложных структур, таких, как массивы или коллекции.

.. code-block:: html+php

    <div class="post">
    <?php

      foreach ($posts as $post) {
        echo "<h1>", $post->title, "</h1>";
      }

    ?>
    </div>

Управление уровнями отрисовки (Rendering Levels)
------------------------------------------------
Как видно выше — :doc:`Phalcon\\Mvc\\View <../api/Phalcon_Mvc_View>` поддерживает иерархию представлений. У вас может возникнуть необходимость в управлении уровнями отрисовки, производимой компонентом представления. Этот функционал предоставляется методом Phalcon\Mvc\\View::setRenderLevel().

Этот метод может быть вызван из контроллера или вышестоящего уровня представления с целью вмешательства в процесс отрисовки.

.. code-block:: php

    <?php

    class PostsController extends \Phalcon\Mvc\Controller
    {

        public function indexAction()
        {

        }

        public function findAction()
        {

            // Ajax-ответ, генерация представления не нужна
            $this->view->setRenderLevel(\Phalcon\Mvc\View::LEVEL_NO_RENDER);

            //...
        }

        public function showAction($postId)
        {
            // Показать только представление, относящееся к конкретному действию контроллера
            $this->view->setRenderLevel(\Phalcon\Mvc\View::LEVEL_ACTION_VIEW);
        }

    }

Допустимые уровни отрисовки:

+-----------------------+--------------------------------------------------------------------------+---------+
| Константы             | Описание                                                                 | Порядок |
+=======================+==========================================================================+=========+
| LEVEL_NO_RENDER       | Указывает, что нужно избежать генерации любых представлений.             |         |
+-----------------------+--------------------------------------------------------------------------+---------+
| LEVEL_ACTION_VIEW     | Генерация представления, относящегося к конкретному действию.            | 1       |
+-----------------------+--------------------------------------------------------------------------+---------+
| LEVEL_BEFORE_TEMPLATE | Генерация шаблонов представлений, предшествующих layout контроллера.     | 2       |
+-----------------------+--------------------------------------------------------------------------+---------+
| LEVEL_LAYOUT          | Генерация представления, для layout контроллера.                         | 3       |
+-----------------------+--------------------------------------------------------------------------+---------+
| LEVEL_AFTER_TEMPLATE  | Генерация шаблонов представлений, следующих за layout контроллера.       | 4       |
+-----------------------+--------------------------------------------------------------------------+---------+
| LEVEL_MAIN_LAYOUT     | Генерация представления для главного layout. Файл views/index.phtml      | 5       |
+-----------------------+--------------------------------------------------------------------------+---------+

Отключение уровней отрисовки
^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Вы можете отключить уровни отрисовки временно или насовсем. Уровень может быть отключен насовсем, если он вообще не используется в приложении:

.. code-block:: php

    <?php

    use Phalcon\Mvc\View;

    $di->set('view', function(){

        $view = new View();

        // Отключить несколько уровней
        $view->disableLevel(array(
            View::LEVEL_LAYOUT => true,
            View::LEVEL_MAIN_LAYOUT => true
        ));

        return $view;

    }, true);

Или временно для какой-либо части приложения:

.. code-block:: php

    <?php

    use Phalcon\Mvc\View;

    class PostsController extends \Phalcon\Mvc\Controller
    {

        public function indexAction()
        {

        }

        public function findAction()
        {
            $this->view->disableLevel(View::LEVEL_MAIN_LAYOUT);
        }

    }

Использование моделей в слое представления
------------------------------------------
Модели приложения всегда доступны из слоя представления. Во время исполнения :doc:`Phalcon\\Loader <../api/Phalcon_Loader>` автоматически создаёт их копии:

.. code-block:: html+php

    <div class="categories">
    <?php

    foreach (Catergories::find("status = 1") as $category) {
       echo "<span class='category'>", $category->name, "</span>";
    }

    ?>
    </div>

Хотя вы и можете вызывать в слое представления такие методы модели, как insert() или update(), это не рекомендуется, так как при этом невозможно передать выполнение другому контроллеру в случае возникновения ошибки или исключения.


Переопределение Представлений (Picking Views)
---------------------------------------------
Как уже упоминалось выше, когда :doc:`Phalcon\\Mvc\\View <../api/Phalcon_Mvc_View>` находится под управлением :doc:`Phalcon\\Mvc\\Application <../api/Phalcon_Mvc_Application>`, тогда отрисовываемым представлением будет какое-то из связанных с последними исполнявшимися контроллером и действием. Это можно переопределить, используя метод Phalcon\\Mvc\\View::pick():

.. code-block:: php

    <?php

    class ProductsController extends \Phalcon\Mvc\Controller
    {

        public function listAction()
        {
            // Использовать в качестве представления для отрисовки "views-dir/products/search"
            $this->view->pick("products/search");
        }

    }

Кэширование фрагментов Представления
------------------------------------
Иногда при разработке динамических веб-сайтов некоторые их области обновляются не так часто. Поэтому результат выполнения похожих запросов так же совпадает. Для увеличения производительности :doc:`Phalcon\\Mvc\\View <../api/Phalcon_Mvc_View>` предоставляет возможность кэширования части или всего отрисованного вывода.

:doc:`Phalcon\\\Mvc\\View <../api/Phalcon_Mvc_View>` используется совместно с :doc:`Phalcon\\Cache <cache>`, чтобы обеспечить простой способ кэширования фрагментов вывода. Вы можете вручную установить обработчик кэша или глобальный обработчик: 

.. code-block:: php

    <?php

    class PostsController extends \Phalcon\Mvc\Controller
    {

        public function showAction()
        {
            // Кэширование с помощью настроек по умолчанию
            $this->view->cache(true);
        }

        public function showArticleAction()
        {
            // Кэширование на один час
            $this->view->cache(array(
                "lifetime" => 3600
            ));
        }

        public function resumeAction()
        {
            // Кэширование представления этого действия на один день с ключем "resume-cache"
            $this->view->cache(
                array(
                    "lifetime" => 86400,
                    "key"      => "resume-cache",
                )
            );
        }

        public function downloadAction()
        {
            // Использование стороннего сервиса для кэширования
            $this->view->cache(
                array(
                    "service"  => "myCache",
                    "lifetime" => 86400,
                    "key"      => "resume-cache",
                )
            );
        }

    }

Если ключ кэша не задан, то компонент автоматически создаёт его используя md5_ для имени представления, отрисовываемого в данный момент.
Это хорошая практика задания ключей кэша для действий, позволяющая идентифицировать кэш, относящийся к конкретному представлению.

Когда компонент Представления должен что-то закэшировать, он запрашивает сервис кэша у контейнера сервисов.
По соглашению, этот сервис именуется как "viewCache":

.. code-block:: php

    <?php

    // Назначение сервиса кэширования представлений
    $di->set('viewCache', function() {

        // Кэширование данных на сутки по умолчанию
        $frontCache = new \Phalcon\Cache\Frontend\Output(array(
            "lifetime" => 86400
        ));

        // Настройки соединения с Memcached
        $cache = new \Phalcon\Cache\Backend\Memcache($frontCache, array(
            "host" => "localhost",
            "port" => "11211"
        ));

        return $cache;
    });

.. highlights::
    Интерфейс всегда должен быть Phalcon\\Cache\\Frontend\\Output, а сервис "viewCache" должен быть зарегистрирован как всегда открытый (not shared)

Использование кэширования представлений также бывает полезно, чтобы предотвратить действия контроллеров, направленные на получение данных, используемых для отображения в представлениях.

Для достижения этой цели необходимо однозначно идентифицировать каждый кэш с помощью ключа. Прежде чем выполнять вычисления или запросы для отображаемых в представлении данных, необходимо убедиться, что кэш не существует или его срок истек:

.. code-block:: html+php

    <?php

    class DownloadController extends \Phalcon\Mvc\Controller
    {

        public function indexAction()
        {

            // Проверяет, кэш с ключом "downloads" на существование или истёкший срок
            if ($this->view->getCache()->exists('downloads')) {

                // Запрос последних загрузок
                $latest = Downloads::find(array(
                    'order' => 'created_at DESC'
                ));

                $this->view->setVar('latest', $latest);
            }

            // Включает кэширование с ключом "downloads"
            $this->view->cache(array(
                'key' => 'downloads'
            ));
        }

    }

Пример реализации кэширования фрагментов — `PHP alternative site`_.

Отключение представления
------------------------
Если в контроллере не производится никакого вывода, то для избежания ненужных обработок можно отключить компонент представления:

.. code-block:: php

    <?php

    class UsersController extends \Phalcon\Mvc\Controller
    {

        public function closeSessionAction()
        {
            // Отключение представления
            $this->view->disable();
        }

    }

Шаблонизаторы
-------------
Шаблонизаторы помогают дизайнерам создавать представления без использования сложного синтаксиса. Phalcon включает в себя мощный и одновременно быстрый шаблонизатор :doc:`Volt <volt>`.

Кроме того, :doc:`Phalcon\\Mvc\\View <../api/Phalcon_Mvc_View>` позволяет использовать другие шаблонизаторы вместо обычного PHP или Volt.

Использование различных шаблонизаторов, как правило, требует сложного разбора кода с применением внешних PHP-библиотек, генерирующих результат для пользователя. Это, в свою очередь, увеличивает количество ресурсов, используемых приложением.

Если используется внешний шаблонизатор, :doc:`Phalcon\\Mvc\\View <../api/Phalcon_Mvc_View>` обеспечивает иерархию файловой структуры и по-прежнему предоставляет доступ к API из этих шаблонов, но с чуть большими затратами.

Этот компонент использует адаптеры, что позволяет Phalcon общаться с внешними шаблонизаторами единым образом. Рассмотрим, как это происходит.

Создание собственного адаптера для шаблонизатора
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Существует множество шаблонизаторов, которые вы можете подключить или создать свой собственный. Первый шаг к использованию внешнего шаблонизатора — это создание адаптера для него.

Адаптер шаблонизатора — это класс, который служит мостом :doc:`Phalcon\\Mvc\\View <../api/Phalcon_Mvc_View>` и самим шаблонизатором. Обычно необходимо реализовать всего два метода: _construct() и render(). В первый передаются экземпляр :doc:`Phalcon\\Mvc\\View <../api/Phalcon_Mvc_View>` и контейнер DI, используемый в приложении.

Во второй — абсолютный путь к файлу представления и параметры, устанавливаемые с помощью $this->view->setVar(). Их можно использовать, как только в них появится необходимость.

.. code-block:: php

    <?php

    class MyTemplateAdapter extends \Phalcon\Mvc\View\Engine
    {

        /**
         * Конструктор адаптера
         *
         * @param \Phalcon\Mvc\View $view
         * @param \Phalcon\DI $di
         */
        public function __construct($view, $di)
        {
            // Инициализация адаптера
            parent::__construct($view, $di);
        }

        /**
         * Отрисовывает представление с помощью шаблонизатора
         *
         * @param string $path
         * @param array $params
         */
        public function render($path, $params)
        {

            // Доступ к view
            $view = $this->_view;

            // Доступ к настройкам
            $options = $this->_options;

            //Render the view
            //...
        }

    }

Изменение шаблонизатора
^^^^^^^^^^^^^^^^^^^^^^^
Вы можете изменить или дополнить шаблонизатор из контроллера следующим образом:

.. code-block:: php

    <?php

    class PostsController extends \Phalcon\Mvc\Controller
    {

        public function indexAction()
        {
            // Назначение шаблонизатора
            $this->view->registerEngines(
                array(
                    ".my-html" => "MyTemplateAdapter"
                )
            );
        }

        public function showAction()
        {
            // Использование нескольких шаблонизаторов
            $this->view->registerEngines(
                array(
                    ".my-html" => 'MyTemplateAdapter',
                    ".phtml" => 'Phalcon\Mvc\View\Engine\Php'
                )
            );
        }

    }

Вы можете полностью заменить шаблонизатор или использовать несколько шаблонизаторов одновременно. Метод \Phalcon\\Mvc\\View::registerEngines() принимает в качестве параметра массив, в котором описываются данные шаблонизаторов. Ключами массива в этом случае будут расширения файлов, что помогает отличить их друг от друга. Файлы шаблонов, относящиеся к этим шаблонизаторам должны иметь соответствующие расширения.

Порядок выполнения шаблонизаторов определяется порядком, в котором они описаны в \Phalcon\\Mvc\\View::registerEngines(). Если :doc:`Phalcon\\Mvc\\View <../api/Phalcon_Mvc_View>` обнаружит два представления с одинаковым именами, но разными расширениями, то он отрисует тот, который был указан первым.

Если вы хотите зарегистрировать шаблонизатор или назначить его для любого запроса в приложении, вы можете сделать это при создании сервиса представления:

.. code-block:: php

    <?php

    // Настройка компонента представления
    $di->set('view', function() {

        $view = new \Phalcon\Mvc\View();

        // A trailing directory separator is required
        $view->setViewsDir('../app/views/');

        $view->registerEngines(array(
            ".my-html" => 'MyTemplateAdapter'
        ));

        return $view;

    }, true);

Адаптеры для некоторых шаблонизаторов можно найти здесь: `Phalcon Incubator <https://github.com/phalcon/incubator/tree/master/Library/Phalcon/Mvc/View/Engine>`_.

Внедрение сервисов в Представление
----------------------------------
Каждое представление, исполняемое внутри экземпляра :doc:`Phalcon\\DI\\Injectable <../api/Phalcon_DI_Injectable>` получает простой доступ к сервисам приложения.

Следующий пример демонстрирует как можно написать `ajax request`_ на jQuery используя url из фреймворка.
Сервис "url" (обычно это :doc:`Phalcon\\Mvc\\Url <url>`) внедрён в представление и доступен как свойство с таким же именем:

.. code-block:: html+php

    <script type="text/javascript">

    $.ajax({
        url: "<?php echo $this->url->get("cities/get") ?>"
    })
    .done(function() {
        alert("Done!");
    });

    </script>

Отдельное использование компонента
----------------------------------
Все компоненты в Phalcon могут быть использованы по-отдельности благодаря их слабой связи друг с другом. Ниже приводится пример самостоятельного использования :doc:`Phalcon\\Mvc\\View <../api/Phalcon_Mvc_View>`:

.. code-block:: php

    <?php

    $view = new \Phalcon\Mvc\View();

    //A trailing directory separator is required
    $view->setViewsDir("../app/views/");

    // Передача переменных в представление
    $view->setVar("someProducts", $products);
    $view->setVar("someFeatureEnabled", true);

    // Начало буферизации вывода
    $view->start();

    // Отрисовка всей иерархии представлений, связанной с products/list.phtml
    $view->render("products", "list");

    // Конец буферизации вывода
    $view->finish();

    echo $view->getContent();

События компонента представлений
--------------------------------
:doc:`Phalcon\\Mvc\\View <../api/Phalcon_Mvc_View>` может отправлять события :doc:`EventsManager <events>`, если последний представлен. Тип событий —  "view". Некоторые из них, возвращая булевое значение false могут остановить текущую операцию. Поддерживаются следующие события:

+----------------------+------------------------------------------------------------+-------------------------------+
| Названия события     | Условия срабатывания                                       | Могут ли остановить операцию? |
+======================+============================================================+===============================+
| beforeRender         | Перед началом процесса отрисовки                           | Да                            |
+----------------------+------------------------------------------------------------+-------------------------------+
| beforeRenderView     | Перед отрисовкой существующего представления               | Да                            |
+----------------------+------------------------------------------------------------+-------------------------------+
| afterRenderView      | После отрисовки существующего представления                | Нет                           |
+----------------------+------------------------------------------------------------+-------------------------------+
| afterRender          | После завершения процесса отрисовки                        | Нет                           |
+----------------------+------------------------------------------------------------+-------------------------------+
| notFoundView         | Если представление не найдено                              | Нет                           |
+----------------------+------------------------------------------------------------+-------------------------------+


Пример ниже демонстрирует как назначить слушателей (listeners) для этого компонента:

.. code-block:: php

    <?php

    $di->set('view', function() {

        // Создание обработчика событий
        $eventsManager = new Phalcon\Events\Manager();

        // Назначение слушателя для событий типа "view"
        $eventsManager->attach("view", function($event, $view) {
            echo $event->getType(), ' - ', $view->getActiveRenderPath(), PHP_EOL;
        });

        $view = new \Phalcon\Mvc\View();
        $view->setViewsDir("../app/views/");

        // Назначение обработчика событий для компонента представления
        $view->setEventsManager($eventsManager);

        return $view;

    }, true);

Следующий пример показывает, как создать плагин, который очищает/исправляет HTML, сгенерированный с использованием Tidy_:

.. code-block:: php

    <?php

    class TidyPlugin
    {

        public function afterRender($event, $view)
        {

            $tidyConfig = array(
                'clean' => true,
                'output-xhtml' => true,
                'show-body-only' => true,
                'wrap' => 0,
            );

            $tidy = tidy_parse_string($view->getContent(), $tidyConfig, 'UTF8');
            $tidy->cleanRepair();

            $view->setContent((string) $tidy);
        }

    }

    // Назначение плагина в качестве слушателя
    $eventsManager->attach("view:afterRender", new TidyPlugin());

.. _this Github repository: https://github.com/bobthecow/mustache.php
.. _ajax request: http://api.jquery.com/jQuery.ajax/
.. _Tidy: http://www.php.net/manual/en/book.tidy.php
.. _md5: http://php.net/manual/en/function.md5.php
.. _PHP alternative site: https://github.com/phalcon/php-site