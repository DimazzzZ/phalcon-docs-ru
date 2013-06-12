Консольные приложения
=====================
CLI приложения выполняются в командной строке. Они часто используются для работы cron'a, скриптов с долгим временем выполнения, командных утилит и т.п.

Задачи
------
Задачи похожи на контроллеры, в них могут быть реализованы действия:

.. code-block:: php

    <?php

    class MonitoringTask extends \Phalcon\CLI\Task
    {

        public function mainAction()
        {

        }

    }

Создание файла запуска
----------------------
Как и в MVC приложениях возможнго создание базового файла запуска

.. code-block:: php

    <?php

   use Phalcon\DI\FactoryDefault\CLI as CliDI,
        Phalcon\CLI\Console as ConsoleApp;

    // Используйте CLI factory в качестве контейнера ресурсов
  $di = new CliDI();

    // Создание консольного приложения
  $console = new ConsoleApp();
  $console->setDI($di);

    // Выполнение действия
  $console->handle(array('task' => 'shell_script_name', 'action' => 'echo'));

