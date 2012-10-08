Class **Phalcon\\Mvc\\Model\\Message**
======================================

Encapsulates validation info generated before save/delete records fails 

.. code-block:: php

    <?php

     use Phalcon\Mvc\Model\Message as Message;
    
     class Robots extends Phalcon\Mvc\Model
    {
    
       public function beforeSave()
       {
         if (this->name == 'Peter') {
            $text = "A robot cannot be named Peter";
            $field = "name";
            $type = "InvalidValue";
            $message = new Message($text, $field, $type);
            $this->appendMessage($message);
         }
       }
    
     }



Methods
---------

public  **__construct** (*string* $message, *string* $field, *string* $type)

Phalcon\\Mvc\\Model\\Message constructor



public  **setType** (*string* $type)

Sets message type



public *string*  **getType** ()

Returns message type



public  **setMessage** (*string* $message)

Sets verbose message



public *string*  **getMessage** ()

Returns verbose message



public  **setField** (*string* $field)

Sets field name related to message



public *string*  **getField** ()

Returns field name related to message



public *string*  **__toString** ()

Magic __toString method returns verbose message



public static :doc:`Phalcon\\Mvc\\Model\\Message <Phalcon_Mvc_Model_Message>`  **__set_state** (*array* $message)

Magic __set_state helps to recover messsages from serialization



