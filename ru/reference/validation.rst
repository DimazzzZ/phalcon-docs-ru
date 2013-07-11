Валидация
=========
Компонент Phalcon\Validation реализует независимую возможность проверки произвольного набора данных.
Компонент можно использовать для проверки данных не относящихся к моделям или коллекциям.

Ниже показан пример использования компонента:

.. code-block:: php

    <?php

    use Phalcon\Validation\Validator\PresenceOf,
        Phalcon\Validation\Validator\Email;

    $validation = new Phalcon\Validation();

    $validation->add('name', new PresenceOf(array(
        'message' => 'The name is required'
    )));

    $validation->add('email', new PresenceOf(array(
        'message' => 'The e-mail is required'
    )));

    $validation->add('email', new Email(array(
        'message' => 'The e-mail is not valid'
    )));

    $messages = $validation->validate($_POST);
    if (count($messages)) {
        foreach ($messages as $message) {
            echo $message, '<br>';
        }
    }

Свободная блочная конструкция этого компонента позволяет вам создавать свои собственные валидаторы 
наряду с теми, что предоставляет фреймворк.

Инициализация валидации
-----------------------
Как вы уже поняли, цепочка проверок может быть иницилизирована простым добавлением их в объект Phalcon\\Validation.
Вы можете повторно использовать код или лучше организовывать проверки размещая их в отдельных файлах:

.. code-block:: php

    <?php

    use Phalcon\Validation,
        Phalcon\Validation\Validator\PresenceOf,
        Phalcon\Validation\Validator\Email;

    class MyValidation extends Validation
    {
        public function initialize()
        {
            $this->add('name', new PresenceOf(array(
                'message' => 'The name is required'
            )));

            $this->add('email', new PresenceOf(array(
                'message' => 'The e-mail is required'
            )));

            $this->add('email', new Email(array(
                'message' => 'The e-mail is not valid'
            )));
        }
    }

.. code-block:: php

    <?php

    $validation = new MyValidation();

    $messages = $validation->validate($_POST);
    if (count($messages)) {
        foreach ($messages as $message) {
            echo $message, '<br>';
        }
    }

Валидаторы
----------
Базовый компонент валидации Phalcon предоставляет следующие правила проверки:

+--------------+-----------------------------------------------------------------------------------------------------------------------------------------+-------------------------------------------------------------------+
| Название     | Описание                                                                                                                                | Пример                                                            |
+==============+=========================================================================================================================================+===================================================================+
| PresenceOf   | Проверяет, что значение поля не равно null или пустой строке.                                                                           | :doc:`Пример <../api/Phalcon_Validation_Validator_PresenceOf>`    |
+--------------+-----------------------------------------------------------------------------------------------------------------------------------------+-------------------------------------------------------------------+
| Identical    | Проверяет, что значение поля соответствует какому-то конкретному значению                                                               | :doc:`Пример <../api/Phalcon_Validation_Validator_Identical>`     |
+--------------+-----------------------------------------------------------------------------------------------------------------------------------------+-------------------------------------------------------------------+
| Email        | Проверяет сответствие формату электронной почты                                                                                         | :doc:`Пример <../api/Phalcon_Validation_Validator_Email>`         |
+--------------+-----------------------------------------------------------------------------------------------------------------------------------------+-------------------------------------------------------------------+
| ExclusionIn  | Проверяет, что значение не входит в список возможных значений                                                                           | :doc:`Пример <../api/Phalcon_Validation_Validator_ExclusionIn>`   |
+--------------+-----------------------------------------------------------------------------------------------------------------------------------------+-------------------------------------------------------------------+
| InclusionIn  | Проверяет, что значение находится в списке возможных значений                                                                           | :doc:`Пример <../api/Phalcon_Validation_Validator_InclusionIn>`   |
+--------------+-----------------------------------------------------------------------------------------------------------------------------------------+-------------------------------------------------------------------+
| Regex        | Проверяет, что значение поля соответствует регулярному выражению                                                                        | :doc:`Пример <../api/Phalcon_Validation_Validator_Regex>`         |
+--------------+-----------------------------------------------------------------------------------------------------------------------------------------+-------------------------------------------------------------------+
| StringLength | Проверяет длину строки                                                                                                                  | :doc:`Пример <../api/Phalcon_Validation_Validator_StringLength>`  |
+--------------+-----------------------------------------------------------------------------------------------------------------------------------------+-------------------------------------------------------------------+
| Between      | Проверяет, что значение находится между двумя другими значениями                                                                        | :doc:`Пример <../api/Phalcon_Validation_Validator_Between>`       |
+--------------+-----------------------------------------------------------------------------------------------------------------------------------------+-------------------------------------------------------------------+
| Confirmation | Проверяет, что значение соответствует другому значению                                                                                  | :doc:`Пример <../api/Phalcon_Validation_Validator_Confirmation>`  |
+--------------+-----------------------------------------------------------------------------------------------------------------------------------------+-------------------------------------------------------------------+

Дополнительные проверки могут быть реализованы самостоятельно. Следующий класс объясняет, как создать правило валидации для этого компонента:

.. code-block:: php

    <?php

    use Phalcon\Validation\Validator,
        Phalcon\Validation\ValidatorInterface,
        Phalcon\Validation\Message;

    class IpValidator extends Validator implements ValidatorInterface
    {

            /**
             * Выполненение валидации
             *
             * @param Phalcon\Validation $validator
             * @param string $attribute
             * @return boolean
             */
            public function validate($validator, $attribute)
            {
                $value = $validator->getValue($attribute);

                if (filter_var($value, FILTER_VALIDATE_URL, FILTER_FLAG_PATH_REQUIRED)) {

                    $message = $this->getOption('message');
                    if (!$message) {
                        $message = 'IP адресс не правилен';
                    }

                    $validator->appendMessage(new Message($message, $attribute, 'Ip'));

                    return false;
                }

                return true;
            }

    }

Важно помнить, что валидаторы возвращают булево значение, показывающее, прошла валидация успешно, либо нет.

Сообщения валидации
-------------------
Компонент :doc:`Phalcon\\Validation <../api/Phalcon_Validation>` имеет внутреннюю подсистему работы с сообщениями.
Она обеспечивает гибкую работу с хранением и выводом проверочных сообщений, генерируемых в ходе проверки.

Каждое сообщение состоит из экземпляра класса :doc:`Phalcon\\Validation\\Message <../api/Phalcon_Mvc_Model_Message>`. Набор
сгенерированных сообщений может быть получен с помощью метода getMessages(). Каждое сообщение содержит расширенную информацию - атрибут,
текст и тип сообщения:

.. code-block:: php

    <?php

    $messages = $validation->validate();
    if (count($messages)) {
        foreach ($validation->getMessages() as $message) {
            echo "Сообщение: ", $message->getMessage(), "\n";
            echo "Поле: ", $message->getField(), "\n";
            echo "Тип: ", $message->getType(), "\n";
        }
    }

Метод getMessages() может быть переопределен в наследуещем классе для замены/перевода текста сообщения по умолчанию,
это особенно актуально для автоматически создаваемых валидаторов:

.. code-block:: php

    <?php

    class MyValidation extends Phalcon\Validation
    {

        public function initialize()
        {
            // ...
        }

        public function getMessages()
        {
            $messages = array();
            foreach (parent::getMessages() as $message) {
                switch ($message->getType()) {
                    case 'PresenceOf':
                        $messages[] = 'Заполнение поля ' . $message->getField() . ' обязательно';
                        break;
                }
            }
            return $messages;
        }
    }

Или вы можете передать сообщение параметром по умолчанию в каждый валидатор:

.. code-block:: php

    <?php

    use Phalcon\Validation\Validator\Email;

    $validation->add('email', new Email(array(
        'message' => 'The e-mail is not valid'
    )));

По умолчанию метод 'getMessages' возвращает все сообщения сгенерированные валидатором. Вы можете отфильтровать 
сообщения используя 'filter':

.. code-block:: php

    <?php

    $messages = $validation->validate();
    if (count($messages)) {
        // Отфильтровать только те сообщения, которые были сгенерированы для поля 'name'
        foreach ($validation->getMessages()->filter('name') as $message) {
            echo $message;
        }
    }

Фильтрация данных
-----------------
Data can be filtering prior to the validation ensuring that malicious data or wrong is not going to
be validated as a proper one.

.. code-block:: php

    <?php

    $validation = new Phalcon\Validation();

    $validation
        ->add('name', new PresenceOf(array(
            'message' => 'The name is required'
        )))
        ->add('email', new PresenceOf(array(
            'message' => 'The email is required'
        )));

    //Filter any extra space
    $validation->setFilters('name', 'trim');
    $validation->setFilters('email', 'trim');

Filtering/Sanitizing is performed using the :doc:`filter <filter>`: component. You can add more filters to this
component or use the built-in ones.

Validation Events
-----------------
When validations are organized in classes, you can implement the methods 'beforeValidation' and 'afterValidation' to
perform additional checks/clean-up etc. If 'beforeValidation' returns 'false' the validation is automatically
cancelled:

.. code-block:: php

    <?php

    use Phalcon\Validation;

    class LoginValidation extends Validation
    {

        public function initialize()
        {
            // ...
        }

        /**
         * Executed before validation
         *
         * @param array $data
         * @param object $entity
         * @param Phalcon\Validation\Message\Group $messages
         */
        public function beforeValidation($data, $entity, $messages)
        {
            if ($this->request->getHttpHost() != 'admin.mydomain.com') {
                $messages->appendMessage(new Message('Users only can log on in the administration domain'));
                return false;
            }
            return true;
        }

        /**
         * Executed after validation
         *
         * @param array $data
         * @param object $entity
         * @param Phalcon\Validation\Message\Group $messages
         */
        public function afterValidation($data, $entity, $messages)
        {
            //... add additional messages or perform more validations
        }

    }

Validation Cancelling
---------------------
By default, all validators assigned to a field are validated regardless if one of them have failed or not. You can change
this behavior by telling the validation component which validator must stop the validation:

.. code-block:: php

    <?php

    use Phalcon\Validation\Validator\PresenceOf,
        Phalcon\Validation\Validator\Regex;

    $validation = new Phalcon\Validation();

    $validation
        ->add('telephone', new PresenceOf(array(
            'message' => 'The telephone is required',
            'cancelOnFail' => true
        )))
        ->add('telephone', new Regex(array(
            'message' => 'The telephone is required',
            'pattern' => '/\+44 [0-9]+/'
        )))
        ->add('telephone', new StringLength(array(
            'minimumMessage' => 'The telephone is too short',
            'min' => 2
        )));

The first validator has the option 'cancelOnFail' => true, therefore if that validator fails the next validator in the chain is not executed.

If you're creating custom validators, you can dynamically stop the validation chain, by setting the 'cancelOnFail' option:

.. code-block:: php

    <?php

    use Phalcon\Validation\Validator,
        Phalcon\Validation\ValidatorInterface,
        Phalcon\Validation\Message;

    class MyValidator extends Validator implements ValidatorInterface
    {

        /**
         * Executes the validation
         *
         * @param Phalcon\Validation $validator
         * @param string $attribute
         * @return boolean
         */
        public function validate($validator, $attribute)
        {
            // If the attribute is name we must stop the chain
            if ($attribute == 'name') {
                $validator->setOption('cancelOnFail', true);
            }

            //...
        }

    }
