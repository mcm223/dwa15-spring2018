# Form.php usage
Form.php is a class which contains form-related helper methods including validation.

[You can download a copy of Form.php here.](https://github.com/susanBuck/dwa15-php/blob/master/includes/Form.php)

Examples of Form.php in use can be found here:
+ Code:
    + <https://github.com/susanBuck/dwa15-php/blob/master/validation.php>
    + <https://github.com/susanBuck/dwa15-php/blob/master/validation-logic.php>
+ In action: 
    + <http://php.dwa15.com/validation.php>


## Known limitations
Form.php is a rudimentary class designed to give us practice with working with external code (specifically OOP code) and form validation.

Given that, it has the following limitations:
+ You can not customize the error messages, nor the name of the field they're reporting on. This is fine for P2.
+ There are only 7 validation rules.

I do not consider Form.php a &ldquo;real world&rdquo; solution&mdash; we're using it for learning/course purposes and to ease us into more sophisticated form operations and validation which we'll see when we get to Laravel. 

(For some context and a preview of what's to come, you can skim the [Laravel docs on validation](https://laravel.com/docs/5.6/validation#available-validation-rules) including the [over 50 different validation rules](https://laravel.com/docs/5.6/validation#available-validation-rules)).


## Installation
Place a copy of [Form.php](https://github.com/susanBuck/dwa15-php/blob/master/includes/Form.php) in your project.


## Usage 
Include `Form.php`; you'll use this class in both your logic and display file, so include it at the top of your display file before your logic file is included.

```php
require('Form.php');
```

In your logic file, instantiate with the $_GET or $_POST request (depending on which method your form is using):
```php
$form = new DWA\Form($_GET);
``` 

## Methods
Extract a value from the request:
```php
$email = $form->get('email');
```

If the value does not exist in the request, this method will return `null`. 

Optionally, you can provide a second parameter to use as a default if the value does not exist, e.g.:

```php
$country = $form->get('country', 'USA');
```

Determine if a value is present in the request:
```php
$email = $form->has('email');
```

Determine if the form submission has occurred:
```php
$isSubmitted = $form->isSubmitted();
```

Validate form data by invoking the `validate` method with an array of fields => validation rules.

```php
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
```

Note that a field can have multiple rules applied, with each rule separated by a `|` character.

The validator will loop through each field's rules, building an array of error messages. Only the first error for a given field will be recorded.

Available validation rules:

+ `required` - Value can not be blank
+ `alphaNumeric` - Value can only contain letters and/or integers
+ `alpha` - Value can only contain letters
+ `numeric` - Value can only contain integers
+ `email` - Value must match a valid email format (eg. `x@y.tld`)
+ `min:x` and/or `max:x` - Value must be greater than and/or less than the given values (x)

(If you wish to add your own validation rules, read this: [Extending Form.php](/php/form.php-extending.md))
 
Example of how you might output the errors in your display file:

```html 
 <?php if ($form->hasErrors) : ?>
     <div class='alert alert-danger'>
         <ul>
             <?php foreach ($errors as $error) : ?>
                 <li><?=$error ?></li>
             <?php endforeach; ?>
         </ul>
     </div>
 <?php endif; ?>
```


Prefill a form input with and without a default value:
```php
<input type='text' name='email' value='<?=$form->prefill("email")?>'>

<input type='text' name='email' value='<?=$form->prefill("email", "example@gmail.com")?>'>
```