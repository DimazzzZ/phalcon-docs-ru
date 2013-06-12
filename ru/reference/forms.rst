Формы
=====
Компонент Phalcon\\Forms позволяет создавать и управлять формами вашего приложения.

Ниже представлен базовый пример работы с формами:

.. code-block:: php

    <?php

    use Phalcon\Forms\Form,
        Phalcon\Forms\Element\Text,
        Phalcon\Forms\Element\Select;

    $form = new Form();

    $form->add(new Text("name"));

    $form->add(new Text("telephone"));

    $form->add(new Select("telephoneType", array(
        'H' => 'Home',
        'C' => 'Cell'
    )));

Элементы форм выводятся по указанным при создании именам:

.. code-block:: html+php

    <h1>Контакты</h1>

    <form method="post">

        <p>
            <label>Имя</label>
            <?php echo $form->render("name") ?>
        </p>

        <p>
            <label>Телефон</label>
            <?php echo $form->render("telephone") ?>
        </p>

        <p>
            <label>Тип телефона</label>
            <?php echo $form->render("telephoneType") ?>
        </p>

        <p>
            <input type="submit" value="Сохранить" />
        </p>

    </form>

Каждый элемент формы может быть настроен по желанию разработчика. Внутри компонент исполльзует возможности
:doc:`Phalcon\\Tag <../api/Phalcon_Tag>` для генерации HTML кода каждого документа, вы можете передавать дополнительные
html-атрибуты вторым параметром:

.. code-block:: html+php

    <p>
        <label>Имя</label>
        <?php echo $form->render("name", array('maxlength' => 30, 'placeholder' => 'Введите своё имя')) ?>
    </p>

Аттрибуты HTML могут быть указаны в параметрах при создании элемента:

.. code-block:: php

    <?php

    $form->add(new Text("name", array(
        'maxlength' => 30,
        'placeholder' => 'Введите своё имя'
    )));


Инициализация
-------------
Как уже говорилось ранее, формы могут быть инициализированы вне форм класса путем добавления элементов к нему. Вы можете повторно использовать
код или организовать формы собранные из разных файлов:

.. code-block:: php

    <?php

    use Phalcon\Forms\Form,
        Phalcon\Forms\Element\Text,
        Phalcon\Forms\Element\Select;

    class ContactForm extends Form
    {
        public function initialize()
        {
            $this->add(new Text("name"));

            $this->add(new Text("telephone"));

            $this->add(new Select("telephoneType", TelephoneTypes::find(), array(
                'using' => array('id', 'name')
            )));
        }
    }


Формы :doc:`Phalcon\\Forms\\Form <../api/Phalcon_Forms_Form>` наследуются от :doc:`Phalcon\\DI\\Injectable <../api/Phalcon_DI_Injectable>`,
предоставляя доступ к службам приложения, если это необходимо:

.. code-block:: php

    <?php

    use Phalcon\Forms\Form,
        Phalcon\Forms\Element\Text,
        Phalcon\Forms\Element\Hidden;

    class ContactForm extends Form
    {

        /**
         * Этот метод возвращает значение по умолчанию для поля 'csrf'
         */
        public function getCsrf()
        {
            return $this->security->getToken();
        }

        public function initialize()
        {

            // Установка сущности
            $this->setEntity($this);

            // Установка поля 'email'
            $this->add(new Text("email"));

            // Добавление скрытого поля csrf
            $this->add(new Hidden("csrf"));
        }
    }

При инициализации формы в конструктор передаётся объект пользователя и другие парамтры:

.. code-block:: php

    <?php

    use Phalcon\Forms\Form,
        Phalcon\Forms\Element\Text,
        Phalcon\Forms\Element\Hidden;

    class UsersForm extends Form
    {
        /**
         * Инициализация формы
         *
         * @param Users $user
         * @param array $options
         */
        public function initialize($user, $options)
        {

            if ($options['edit']) {
                $this->add(new Hidden('id'));
            } else {
                $this->add(new Text('id'));
            }

            $this->add(new Text('name'));
        }
    }

Теперь можно использовать экземпляр формы:

.. code-block:: php

    <?php

    $form = new UsersForm(new Users(), array('edit' => true));

Валидация
---------
Формы в Phalcon интегрированы с компонентом :doc:`валидации <validation>` для быстрой проверки введённых данных. Для каждого элемента формы можно
устанавливать готовый или настраиваемый валидатор:

.. code-block:: php

    <?php

    use Phalcon\Forms\Element\Text,
        Phalcon\Validation\Validator\PresenceOf,
        Phalcon\Validation\Validator\StringLength;

    $name = new Text("name");

    $name->addValidator(new PresenceOf(array(
        'message' => 'Поле Name обязательно для заполнения'
    )));

    $name->addValidator(new StringLength(array(
        'min' => 10,
        'messageMinimum' => 'Значение поля Name слишком короткое'
    )));

    $form->add($name);

Затем вы сможете проверить правильность заполнения формы пользователем:

.. code-block:: php

    <?php

    if (!$form->isValid($_POST)) {
        foreach ($form->getMessages() as $message) {
            echo $message, '<br>';
        }
    }

Валидаторы выполняются в порядке регистрации.

По умолчанию сообщения, генерируемые всеми элементами формы объединены, чтобы их можно было собрать одним проходом foreach,
вы можете изменить это поведение, чтобы получить сообщения, разделенные по типам:

.. code-block:: php

    <?php

    foreach ($form->getMessages(false) as $attribute => $messages) {
        echo 'Сообщение создано ', $attribute, ':', "\n";
        foreach ($messages as $message) {
            echo $message, '<br>';
        }
    }


Так же можно получить сообщения конкретного элемента:

.. code-block:: php

    <?php

    foreach ($form->getMessagesFor('name') as $message) {
        echo $message, '<br>';
    }


Фильтрация
----------
A form is also able to filter data before be validated, you can set filters in each element:



Настройка пользовательских параметров
-------------------------------------
Формы и сущности
----------------
Модели или коллекции являются такими сущностями, которые можно без проблем связать с формами, их значения в таком случае будут использоваться
по умолчанию для соответствующих по именам значений элементов форм. Всё это делается очень легко:

.. code-block:: php

    <?php

    $robot = Robots::findFirst();

    $form = new Form($robot);

    $form->add(new Text("name"));

    $form->add(new Text("year"));

При отображении формы, если нет значений по умолчанию для элементов, будут использованы значения из сущностей:

.. code-block:: html+php

    <?php echo $form->render('name') ?>

Проверить введённые пользователем значения в форму можно следующим образом:

.. code-block:: php

    <?php

    $form->bind($_POST, $robot);

    // Проверка правильности введённых данных формы
    if ($form->isValid()) {

        // Сохранение сущности
        $robot->save();
    }

Setting up a plain class as entity also is possible:

.. code-block:: php

    <?php

    class Preferences
    {

        public $timezone = 'Europe/Amsterdam';

        public $receiveEmails = 'No';

    }

Using this class as entity, allows the form to take the default values from it:

.. code-block:: php

    <?php

    $form = new Form(new Preferences());

    $form->add(new Select("timezone", array(
        'America/New_York' => 'New York',
        'Europe/Amsterdam' => 'Amsterdam',
        'America/Sao_Paulo' => 'Sao Paulo',
        'Asia/Tokio' => 'Tokio',
    )));

    $form->add(new Select("receiveEmails", array(
        'Yes' => 'Yes, please!',
        'No' => 'No, thanks'
    )));

Entities can implement getters, which have more precedence than public propierties, these methods
give you more free to produce values:

.. code-block:: php

    <?php

    class Preferences
    {

        public $timezone;

        public $receiveEmails;

        public function getTimezone()
        {
            return 'Europe/Amsterdam';
        }

        public function getTimezone()
        {
            return 'No';
        }

    }

Элементы форм
-------------
Phalcon предоставляет набор элементов для использования в ваших формах:

+--------------+-------------------------------------------------------------------+---------------------------------------------------------+
| Название     | Описание                                                          | Пример использования                                    |
+==============+===================================================================+=========================================================+
| Text         | Генерирует элемент INPUT[type=text]                               | :doc:`Пример <../api/Phalcon_Forms_Element_Text>`       |
+--------------+-------------------------------------------------------------------+---------------------------------------------------------+
| Password     | Генерирует элемент INPUT[type=password]                           | :doc:`Пример <../api/Phalcon_Forms_Element_Password>`   |
+--------------+-------------------------------------------------------------------+---------------------------------------------------------+
| Select       | Генерирует элемент раскрывающегося списка SELECT                  | :doc:`Пример <../api/Phalcon_Forms_Element_Select>`     |
+--------------+-------------------------------------------------------------------+---------------------------------------------------------+
| Check        | Генерирует элемент INPUT[type=check]                              | :doc:`Пример <../api/Phalcon_Forms_Element_Check>`      |
+--------------+-------------------------------------------------------------------+---------------------------------------------------------+
| Textarea     | Генерирует элемент TEXTAREA                                       | :doc:`Пример <../api/Phalcon_Forms_Element_TextArea>`   |
+--------------+-------------------------------------------------------------------+---------------------------------------------------------+
| Hidden       | Генерирует элемент INPUT[type=hidden]                             | :doc:`Пример <../api/Phalcon_Forms_Element_Hidden>`     |
+--------------+-------------------------------------------------------------------+---------------------------------------------------------+
| File         | Генерирует элемент INPUT[type=file]                               | :doc:`Пример <../api/Phalcon_Forms_Element_File>`       |
+--------------+-------------------------------------------------------------------+---------------------------------------------------------+
| Date         | Генерирует элемент INPUT[type=date]                               | :doc:`Пример <../api/Phalcon_Forms_Element_Date>`       |
+--------------+-------------------------------------------------------------------+---------------------------------------------------------+
| Numeric      | Генерирует элемент INPUT[type=number]                             | :doc:`Пример <../api/Phalcon_Forms_Element_Numeric>`    |
+--------------+-------------------------------------------------------------------+---------------------------------------------------------+
| Submit       | Генерирует элемент INPUT[type=submit]                             | :doc:`Пример <../api/Phalcon_Forms_Element_Submit>`     |
+--------------+-------------------------------------------------------------------+---------------------------------------------------------+

Event Callbacks
---------------
Whenever forms are implemented as classes, the callbacks: beforeValidation and afterValidation can be implemented
in the form's class to perform pre-validations and post-validations:

.. code-block:: html+php

    <?php

    class ContactForm extends Phalcon\Mvc\Form
    {
        public function beforeValidation()
        {

        }
    }

Rendering Forms
---------------
You can render the form with total flexibility, the following example shows how to render each element using an standard procedure:

.. code-block:: html+php

    <?php

    <form method="post">
        <?php
            //Traverse the form
            foreach ($form as $element) {

                //Get any generated messages for the current element
                $messages = $form->getMessagesFor($element->getName());

                if (count($messages)) {
                    //Print each element
                    echo '<div class="messages">';
                    foreach ($messages as $message) {
                        echo $message;
                    }
                    echo '</div>';
                }

                echo '<p>';
                echo '<label for="', $element->getName(), '">', $element->getLabel(), '</label>';
                echo $element;
                echo '</p>';

            }
        ?>
        <input type="submit" value="Send"/>
    </form>

Or reuse the logic in your form class:

.. code-block:: php

    <?php

    class ContactForm extends Phalcon\Forms\Form
    {
        public function initialize()
        {
            //...
        }

        public function renderDecorated($name)
        {
            $element = $this->get($name);

            //Get any generated messages for the current element
            $messages = $this->getMessagesFor($element->getName());

            if (count($messages)) {
                //Print each element
                echo '<div class="messages">';
                foreach ($messages as $message) {
                    echo $this->flash->error($message);
                }
                echo '</div>';
            }

            echo '<p>';
            echo '<label for="', $element->getName(), '">', $element->getLabel(), '</label>';
            echo $element;
            echo '</p>';
        }

    }

In the view:

.. code-block:: php

    <?php

    echo $element->renderDecorated('name');

    echo $element->renderDecorated('telephone');

Creating Form Elements
----------------------
In addition to the form elements provided by Phalcon you can create your own custom elements:

.. code-block:: php

    <?php

    use Phalcon\Forms\Element;

    class MyElement extends Element
    {
        public function render($attributes=null)
        {
            $html = //... produce some html
            return $html;
        }
    }

Forms Manager
-------------
This component provides a forms manager that can be used by the developer to register forms and access them via the service locator:

.. code-block:: php

    <?php

    $di['forms'] = function() {
        return new Phalcon\Forms\Manager();
    }

Forms are added to the forms manager and referenced by a unique name:

.. code-block:: php

    <?php

    $this->forms->set('login', new LoginForm());

Using the unique name, forms can be accesed in any part of the application:

.. code-block:: php

    <?php

    echo $this->forms->get('login')->render();

Внешние источники
-----------------
* `Vökuró <http://vokuro.phalconphp.com>`_, is a sample application that uses the forms builder to create forms in this application, [`Github <https://github.com/phalcon/vokuro>`_]
