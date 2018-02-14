# Extending Form.php
If you want to add new validation methods or overwrite the behavior of existing validation methods, you can create your own Form class that builds upon the provided Form class, demonstrating a key concept of OOP programming known as **inheritance**. 

## Step 1 
Create a new class which will extend the behavior of the provided `Form` class. The following example shows a demo with a new class called `MyForm`.

Note how `MyForm` __extends__ the parent `Form` class allowing it to inherit all of `Form`'s public or protected methods and properties.

In this case, I've defined a new `numeric` function that will overwrite the `numeric` function found in the parent Form class (the new version allows numbers with decimal places). You can overwrite existing validation functions or define new ones, depending on what your needs are.

```php
<?php
# Filename: MyForm.php

namespace Buck; # Choose your own namespace for this class. Could be your last name, your app name, etc.

class MyForm extends DWA\Form
{

    /**
     * Returns boolean if given value contains only numbers
     */
    protected function numeric($value)
    {
        # Check if value (sans decimals) is numeric
        $numeric = ctype_digit(str_replace([' ', '.'], '', $value));

        # A valid number should have either 0 or 1 decimals
        $oneOrNoneDecimal = in_array(substr_count($value, '.'), [0, 1]);

        return $numeric and $oneOrNoneDecimal;
    }
}
```

## Step 2
Update your logic file so that it includes your new form class (`MyForm`) and instantiates it instead of `Form.`

```php
<?php
require('Form.php');
require('MyForm.php'); # <--- NEW

$form = new Buck\MyForm($_GET); # <--- CHANGED

if ($form->isSubmitted()) {
    $errors = $form->validate(
        [
            'email' => 'required|email',
            'username' => 'alphaNumeric',
            'year' => 'numeric',
            'age' => 'min:16',
            'score' => 'max:5',
            'rank' => 'numeric|min:0|max:5',
        ]
    );
}
```
