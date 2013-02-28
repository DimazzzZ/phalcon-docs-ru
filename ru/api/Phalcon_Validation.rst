Class **Phalcon\\Validation**
=============================




Methods
---------

public  **validate** ([*array|object* $data])

Validate a set of data according to a set of rules



public *Phalcon\\Validator*  **add** (*string* $attribute, *unknown* $validator)

Adds a validator to a field



public *array*  **getValidators** ()

Returns the registered validators



public *array*  **getMessages** ()

Returns the validation messages produced by the validation process



public  **appendMessage** (*Phalcon\\Validation\\MessageInterface* $message)

Appends a message to the messages list



public *Phalcon\\Validator*  **bind** (*array* $data)

Assigns the data to an entity The entity is used to obtain the validation values



public *mixed*  **getValue** (*unknown* $key)

Obtain an value from the data passed to the validator If an entity is



