Логирование
===========
:doc:`Phalcon\\Logger <../api/Phalcon_Logger>` является компонентом для обеспечиния ведения логов в приложении. Он позволяет
вести разных типов с использованием различных адаптеров. Он также предлагает регистрацию транзакций, параметров конфигурации, различных форматов и фильтров.
Вы можете использовать :doc:`Phalcon\\Recorder <../api/Phalcon_Logger>` для логирования всех операций, отладки процессов и отслеживания работы приложения.

Адаптеры
--------
Этот компонент позволяет использовать адаптеры для хранения журнала сообщений. Использование адаптеров обеспечивает общий интерфейс для регистрации
время переключения интерфейсов при необходимости. Реализованные адаптеры:

+---------+-------------------------------+--------------------------------------------------------------------------------+
| Адаптер | Описание                      | API                                                                            |
+=========+===============================+================================================================================+
| File    | Логирование в текстовой файл  | :doc:`Phalcon\\Logger\\Adapter\\File <../api/Phalcon_Logger_Adapter_File>`     |
+---------+-------------------------------+--------------------------------------------------------------------------------+
| Stream  | Логирование в PHP поток       | :doc:`Phalcon\\Logger\\Adapter\\Stream <../api/Phalcon_Logger_Adapter_Stream>` |
+---------+-------------------------------+--------------------------------------------------------------------------------+
| Syslog  | Логирование в системный журнал| :doc:`Phalcon\\Logger\\Adapter\\Syslog <../api/Phalcon_Logger_Adapter_Syslog>` |
+---------+-------------------------------+--------------------------------------------------------------------------------+

Создание журнала
----------------
В приведенном ниже примере показано, как создать журнал и добавить в него запись:

.. code-block:: php

    <?php

    $logger = new \Phalcon\Logger\Adapter\File("app/logs/test.log");
    $logger->log("This is a message");
    $logger->log("This is an error", \Phalcon\Logger::ERROR);
    $logger->error("This is another error");

Результат кода:

.. code-block:: php

    [Tue, 17 Apr 12 22:09:02 -0500][DEBUG] This is a message
    [Tue, 17 Apr 12 22:09:02 -0500][ERROR] This is an error
    [Tue, 17 Apr 12 22:09:02 -0500][ERROR] This is another error

Транзакции
----------
Запись данных в адаптер т.е. в файл (файловая система) всегда является 'дорогостоющей' операцией с точки зрения производительности. 
Для решения этой задачи, можно использовать транзакции при логировании. Транзакции временно хронят записи в памяти, а затем переносят их
соответствующий адаптер (в данном случае в файл).

.. code-block:: php

    <?php

    // Создание логгера
    $logger = new \Phalcon\Logger\Adapter\File("app/logs/test.log");

    // Начало транзакции
    $logger->begin();

    // Добавление записей
    $logger->alert("This is an alert");
    $logger->error("This is another error");

    // Размещение записей в файл
    $logger->commit();

Одновременное логирование нескольких обработчиков
-------------------------------------------------
:doc:`Phalcon\\Logger <../api/Phalcon_Logger>` позволяет отправку сообщений на несколько обработчиков одним вызовом:

.. code-block:: php

    <?php

    $logger = new \Phalcon\Logger\Multiple();

    $logger->push(new \Phalcon\Logger\Adapter\File('test.log'));
    $logger->push(new \Phalcon\Logger\Adapter\Stream('php://stdout'));

    $logger->log("This is a message");
    $logger->log("This is an error", \Phalcon\Logger::ERROR);
    $logger->error("This is another error");

Сообщения отправляются на обработчик в порядке их регистраций.

Форматирование сообщений
------------------------
Данный компонент позволяет использовать 'formatters' для форматирования сообщений перед тем как их отправить на бэкенд. 
Реализованные следующие форматеры:

+---------+--------------------------------------------------+------------------------------------------------------------------------------------+
| Адаптер | Описание                                         | API                                                                                |
+=========+==================================================+====================================================================================+
| Line    | Оформление записей одной строкой                 | :doc:`Phalcon\\Logger\\Formatter\\Line <../api/Phalcon_Logger_Formatter_Line>`     |
+---------+--------------------------------------------------+------------------------------------------------------------------------------------+
| Json    | Подготовка записей для преобразoвание в JSON     | :doc:`Phalcon\\Logger\\Formatter\\Json <../api/Phalcon_Logger_Formatter_Json>`     |
+---------+--------------------------------------------------+------------------------------------------------------------------------------------+
| Syslog  | Подготовка записи для отправки в системный журнал| :doc:`Phalcon\\Logger\\Formatter\\Syslog <../api/Phalcon_Logger_Formatter_Syslog>` |
+---------+--------------------------------------------------+------------------------------------------------------------------------------------+

Линейный Оформитель
^^^^^^^^^^^^^^^^^^^
Оформление записей в одну строку. Формат по умолчнию:

[%date%][%type%] %message%

You can change the default format using setFormat(), this allows you to change the format of the logged
messages by defining your own. The log format variables allowed are:

+-----------+------------------------------------------+
| Переменные| Описание                                 |
+===========+==========================================+
| %message% | Запись которая будет внесена            |
+-----------+------------------------------------------+
| %date%    | Дата добавления записи в журнал          |
+-----------+------------------------------------------+
| %type%    | Тип записи заглавными буквами            |
+-----------+------------------------------------------+

В приведенном примере показанно как изменить формат лога:

.. code-block:: php

    <?php

    //Changing the logger format
    $formatter = new Phalcon\Logger\Formatter\Line("%date% - %message%");
    $logger->setFormatter($formatter);

Реализация собственного оформителя
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
The :doc:`Phalcon\\Logger\\FormatterInterface <../api/Phalcon_Logger_FormatterInterface>` interface must be implemented in order to
create your own logger formatter or extend the existing ones.

Adapters
--------
The following examples show the basic use of each adapter:

Stream Logger
^^^^^^^^^^^^^
The stream logger writes messages to a valid registered stream in PHP. A list of streams is available `here <http://php.net/manual/en/wrappers.php>`_:

.. code-block:: php

    <?php

    // Opens a stream using zlib compression
    $logger = new \Phalcon\Logger\Adapter\Stream("compress.zlib://week.log.gz");

    // Writes the logs to stderr
    $logger = new \Phalcon\Logger\Adapter\Stream("php://stderr");

File Logger
^^^^^^^^^^^
This logger uses plain files to log any kind of data. By default all logger files are open using
append mode which open the files for writing only; placing the file pointer at the end of the file.
If the file does not exist, attempt to create it. You can change this mode passing additional options to the constructor:

.. code-block:: php

    <?php

    // Create the file logger in 'w' mode
    $logger = new \Phalcon\Logger\Adapter\File("app/logs/test.log", array(
        'mode' => 'w'
    ));

Syslog Logger
^^^^^^^^^^^^^
This logger sends messages to the system logger. The syslog behavior may vary from one operating system to another.

.. code-block:: php

    <?php

    // Basic Usage
    $logger = new \Phalcon\Logger\Adapter\Syslog(null);

    // Setting ident/mode/facility
    $logger = new \Phalcon\Logger\Adapter\Syslog("ident-name", array(
        'option' => LOG_NDELAY,
        'facility' => LOG_MAIL
    ));

Implementing your own adapters
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
The :doc:`Phalcon\\Logger\\AdapterInterface <../api/Phalcon_Logger_AdapterInterface>` interface must be implemented in order to
create your own logger adapters or extend the existing ones.
