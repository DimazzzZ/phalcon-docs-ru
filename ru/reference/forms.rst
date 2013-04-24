Формы
=====
Компонент Phalcon\Forms позволяет создавать и управлять формами вашего приложения.

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

	class ContactsForm extends Form
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

Элементы форм
-------------
Phalcon предоставляет набор элементов для использования в ваших формах:

+--------------+-------------------------------------------------------------------+-------------------------------------------------------------------+
| Название     | Описание                                                          | Example                                                           |
+==============+===================================================================+===================================================================+
| Text         | Генерирует элемент INPUT[type=text]                               | :doc:`Example <../api/Phalcon_Forms_Element_Text>`                |
+--------------+-------------------------------------------------------------------+-------------------------------------------------------------------+
| Password     | Генерирует элемент INPUT[type=password]                           | :doc:`Example <../api/Phalcon_Forms_Element_Password>`            |
+--------------+-------------------------------------------------------------------+-------------------------------------------------------------------+
| Select       | Генерирует элемент раскрывающегося списка SELECT                  | :doc:`Example <../api/Phalcon_Forms_Element_Select>`              |
+--------------+-------------------------------------------------------------------+-------------------------------------------------------------------+
| Radio        | Генерирует элемент INPUT[type=radio]                              | :doc:`Example <../api/Phalcon_Forms_Element_Radio>`               |
+--------------+-------------------------------------------------------------------+-------------------------------------------------------------------+
| Check        | Генерирует элемент INPUT[type=check]                              | :doc:`Example <../api/Phalcon_Forms_Element_Check>`               |
+--------------+-------------------------------------------------------------------+-------------------------------------------------------------------+
| Textarea     | Генерирует элемент TEXTAREA                                       | :doc:`Example <../api/Phalcon_Forms_Element_TextArea>`            |
+--------------+-------------------------------------------------------------------+-------------------------------------------------------------------+
