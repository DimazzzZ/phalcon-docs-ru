Сохранение данных в сессии
==========================

Компонент :doc:`Phalcon\\Session <../api/Phalcon_Session>` предоставляет объектно-ориентированный интерфейс для работы с сессиями.

Запуск сессий
-------------
Некоторые приложения активно используют в своей работе сессии, используя их в каждом действии. Другие наоборот, используют сессии мало и не часто.
Благодаря использованию контейнера сервисов, мы можем гарантировать что запуск сессий будет произведён только по необходимости.

.. code-block:: php

    <?php

    // Сессии запустятся один раз, при первом обращении к объекту
    $di->setShared('session', function() {
        $session = new Phalcon\Session\Adapter\Files();
        $session->start();
        return $session;
    });

Сохранение/получение данных из сессий
-------------------------------------
Из контроллера, представления (view) или другого компонента расширяющего :doc:`Phalcon\\DI\\Injectable <../api/Phalcon_DI_Injectable>` можно
получить доступ к сессиям и работать с ними следующим образом:

.. code-block:: php

    <?php

    class UserController extends Phalcon\Mvc\Controller
    {

        public function indexAction()
        {
            // Установка значения сессии
            $this->session->set("user-name", "Michael");
        }

        public function welcomeAction()
        {

            // Проверка наличия переменной сессии
            if ($this->session->has("user-name")) {

                // Получение значения
                $name = $this->session->get("user-name");
            }
        }

    }

Удаление/очистка сессий
-----------------------
Таким же способом пожно удалить перевенную сессии, или целиком всё очистить:

.. code-block:: php

    <?php

    class UserController extends Phalcon\Mvc\Controller
    {

        public function removeAction()
        {
            // Удаление переменной сессии
            $this->session->remove("user-name");
        }

        public function logoutAction()
        {
            // Полная очистка сессии
            $this->session->destroy();
        }

    }

Изоляция данных сессии внутри приложения
----------------------------------------
Иногда пользователь может запускать одно и тоже приложение несколкьо раз, на одном и том же сервере, в одно время. Естественно, используя
переменные сессий нам бы хотелось  что бы все приложения получали получали доступ к разным сессиям (хотя в одинаковых приложениях и код одинаковый и названия переменных).
Для решения этой проблемы можно использовать преффикс для переменных сессий, разный для разных приложений.
.. code-block:: php

    <?php

    // Изоляция данных сессий
    $di->set('session', function(){

        // Все переменные этого приложения будет иметь преффикс "my-app-1"
        $session = new Phalcon\Session\Adapter\Files(
            array(
                'uniqueId' => 'my-app-1'
            )
        );

        $session->start();

        return $session;
    });

На работе это никак не скажется, добавлять преффикс вручную во время установки или чтения сессий нет необходимости.

Сессионные Bags
---------------
Компонент :doc:`Phalcon\\Session\\Bag <../api/Phalcon_Session_Bag>` позволяет работать с сессиями разделяя их по пространствам имён.
Работая таким образом, вы можете легко создавать группы переменных сессии в приложении. Установив значение переменной такого объекта,
оно автоматически сохранится в сессии:

.. code-block:: php

    <?php

    $user       = new Phalcon\Session\Bag();
    $user->name = "Kimbra Johnson";
    $user->age  = 22;


Сохранение данных в компонентах
-------------------------------
Контроллеры, компоненты и классы расширяющие :doc:`Phalcon\\DI\\Injectable <../api/Phalcon_DI_Injectable>` могут работать
с :doc:`Phalcon\\Session\\Bag <../api/Phalcon_Session_Bag>` напрямую. Компонент в таком случае изолирует данные для каждого класса.
Благодаря этому вы можете сохранять данные между запросами используя единообразные пути.

.. code-block:: php

    <?php

    class UserController extends Phalcon\Mvc\Controller
    {

        public function indexAction()
        {
            // Создаётся постоянная (persistent) переменная "name"
            $this->persistent->name = "Laura";
        }

        public function welcomeAction()
        {
            if (isset($this->persistent->name))
            {
                echo "Привет, ", $this->persistent->name;
            }
        }

    }

И в компоненте:

.. code-block:: php

    <?php

    class Security extends Phalcon\Mvc\User\Component
    {

        public function auth()
        {
            // Создаётся постоянная (persistent) переменная "name"
            $this->persistent->name = "Laura";
        }

        public function getAuthName()
        {
            return $this->persistent->name;
        }

    }

Данные, добавленные непосредственно в сессию ($this->session) доступны во всём приложении, в то время как persistent ($this->persistent)
переменные доступны только внутри своего текущего класса.

Реализация собственных адаптеров сессий
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
Для создания адаптера необходимо реализовать интерфейс :doc:`Phalcon\\Session\\AdapterInterface <../api/Phalcon_Session_AdapterInterface>`,
или использовать наследование от готового с доработкой необходимой логики.

У нас есть некоторые готовые адаптеры для сессий `Phalcon Incubator <https://github.com/phalcon/incubator/tree/master/Library/Phalcon/Session/Adapter>`_
