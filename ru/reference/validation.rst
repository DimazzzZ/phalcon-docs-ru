Валидация
=========
Компонент Phalcon\Validation реализует независимую возможность проверки произвольного набора данных.
Компонет можно использовать для проверки данных не относящихся к моделям или коллекциям.

Ниже показан пример использования компонента:

.. code-block:: php

		<?php
		
		use Phalcon\Validation\Validator\PresenceOf,
			Phalcon\Validation\Validator\Email;
		
		$validation = new Phalcon\Validation();
		
		$validation->add('name', new PresenceOf(
			array('message' => 'The name is required')
		));
		
		$validation->add('email', new PresenceOf(
			array('message' => 'The e-mail is required')
		));
		
		$validation->add('email', new Email(
			array('message' => 'The e-mail is not valid')
		));
		
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
| PresenceOf   | Проверяет, что значение поля не равно null или пустой строке.                                                                           | :doc:`Example <../api/Phalcon_Validation_Validator_PresenceOf>`   |
+--------------+-----------------------------------------------------------------------------------------------------------------------------------------+-------------------------------------------------------------------+
| Email        | Проверяет сответствие email формату                                                                                                     | :doc:`Example <../api/Phalcon_Validation_Validator_Email>`        |
+--------------+-----------------------------------------------------------------------------------------------------------------------------------------+-------------------------------------------------------------------+
| ExclusionIn  | Проверяет, что значение не входит в список возможных значений                                                                           | :doc:`Example <../api/Phalcon_Validation_Validator_ExclusionIn>`  |
+--------------+-----------------------------------------------------------------------------------------------------------------------------------------+-------------------------------------------------------------------+
| InclusionIn  | Проверяет, что значение находится в списке возможных значений                                                                           | :doc:`Example <../api/Phalcon_Validation_Validator_InclusionIn>`  |
+--------------+-----------------------------------------------------------------------------------------------------------------------------------------+-------------------------------------------------------------------+
| Regex        | Проверяет, что значение поля соответствует регулярному выражению                                                                        | :doc:`Example <../api/Phalcon_Validation_Validator_Regex>`        |
+--------------+-----------------------------------------------------------------------------------------------------------------------------------------+-------------------------------------------------------------------+
| StringLength | Проверяет длину строки                                                                                                                  | :doc:`Example <../api/Phalcon_Validation_Validator_StringLength>` |
+--------------+-----------------------------------------------------------------------------------------------------------------------------------------+-------------------------------------------------------------------+

Дополнительные проверки могут быть реализованы самостоятельно. Следующий класс, объясняет, как создать правило валидации для этого компонента:

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

	$validation->add('email', new Phalcon\Validation\Validator\Email(
		'message' => 'Адресс e-mail введён не верно'
	));



