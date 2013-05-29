Управление ресурсами (Assets Management)
========================================
Phalcon\Assets - это компонент позволяющий разработчику управлять статичными ресурсами в веб-приложении,
такими как каскадные таблицы стилей или javascript'ы.

:doc:`Phalcon\\Assets\\Manager <../api/Phalcon_Assets_Manager>` доступен в контейнере сервисов,
т.ч. вы можете добавлять ресурсы из любой части приложения где он доступен.

Добавление ресурсов
-------------------
Поддерживаются ресурсы двух типов: каскадные таблицы стилей и javascript'ы, но при необходимости,
можете создать и другие. Внтуренний механизм менеджера ресурсов хранит две коллекции, одну
для javascript'ов, а другую для каскадных таблиц стилей.

Добавить ресурсы в эти коллекции очень просто:

.. code-block:: php

    <?php

    class IndexController extends Phalcon\Mvc\Controller
    {
        public function index()
        {

            // Добавляем некоторые локальные таблицы стилей
            $this->assets
                ->addCss('css/style.css')
                ->addCss('css/index.css');

            // и javascript'ы
            $this->assets
                ->addJs('js/jquery.js')
                ->addJs('js/bootstrap.min.js');

        }
    }

Далее, добавленные ресурсы могут быть отображены в представлениях:

.. code-block:: html+php

    <html>
        <head>
            <title>Некоторый удивительный веб-сайт</title>
            <?php $this->assets->outputCss() ?>
        </head>
        <body>

            <!-- ... -->

            <?php $this->assets->outputJs() ?>
        </body>
    <html>

Локальные/удаленные ресурсы
---------------------------
Локальные ресурсы это те, которые предоставляются вами в том же приложении.
Ссылки на локальные ресурсы генерируются с помощью сервиса 'url', чаще
с применением :doc:`Phalcon\\Mvc\\Url <../api/Phalcon_Mvc_Url>`.

Удаленные ресурсы, такие как общие библиотеки, на подобии jquery, bootstrap или пр. предоставляемые посредством CDN.

.. code-block:: php

    <?php

    // Добавляем некоторые локальные и удаленные ресурсы
    $this->assets
        ->addCss('//netdna.bootstrapcdn.com/twitter-bootstrap/2.3.1/css/bootstrap-combined.min.css', false)
        ->addCss('css/style.css', true);

Коллекции
---------
В коллекциях группируются однотипные ресурсы. Менеджер ресурсов безоговорочно создает две: css и js.
Для группирования специфичных ресурсов вы можете создавать дополнительные:

.. code-block:: php

    <?php

    // Javascript'ы в заголовке
    $this->assets
        ->collection('header')
        ->addJs('js/jquery.js')
        ->addJs('js/bootstrap.min.js');

    // Javascript'ы в "подвале"
    $this->assets
        ->collection('footer')
        ->addJs('js/jquery.js')
        ->addJs('js/bootstrap.min.js');

затем в представлении:

.. code-block:: html+php

    <html>
        <head>
            <title>Некоторый удивительный веб-сайт</title>
            <?php $this->assets->outputJs('header') ?>
        </head>
        <body>

            <!-- ... -->

            <?php $this->assets->outputJs('footer') ?>
        </body>
    <html>

Префиксы
--------
К коллекциям могут применятся URL префиксы, это позволит в любой момент легко изменить расположение ресурсов с одного сервера на другой:

.. code-block:: php

    <?php

    $scripts = $this->assets->collection('footer');

    if ($config->enviroment == 'development') {
        $scripts->setPrefix('/');
    } else {
        $scripts->setPrefix('http:://cdn.example.com/');
    }

    $scripts->addJs('js/jquery.js')
            ->addJs('js/bootstrap.min.js');

Также, доступен синтаксис цепочки (chainable):

.. code-block:: php

    <?php

    $scripts = $assets
        ->collection('header')
        ->setPrefix('http:://cdn.example.com/')
        ->setLocal(false)
        ->addJs('js/jquery.js')
        ->addJs('js/bootstrap.min.js');

Пользовательский вывод
----------------------
Методы outputJs и outputCss создают требуемую HTML-разметку в соответствии с каждым типом ресурсов, но
вы можете переопределить эти методы и создать разметку вручную:

.. code-block:: php

    <?php

    foreach ($this->assets->collection('js') as $resource) {
        echo Phalcon_Tag::javascriptInclude($resource->getPath());
    }
