# Foobooks: One to Many

The following is a __rough outline__ of some of the modifications I'll make to Foobooks during Week 13.

__This should not be considered a stand-alone document; for full details please refer to the lecture video and the Foobooks code source.__


## Author dropdown
To associate Authors with Books we'll use a dropdown filled with authors. We'll start by showing this in the *Add* feature, and later add it to the *Edit* feature.

To make this happen, we first need an __array of authors__ where the key is the author `id` and the value is the author `name`.

```php
# BookController.php
public function create($id)
{
    # Get all the authors
    $authors = Author::orderBy('last_name')->get();

    # Organize the authors into an array where the key = author id and value = author name
    $authorsForDropdown = [];
    foreach ($authors as $author) {
        $authorsForDropdown[$author->id] = $author->last_name.', '.$author->first_name;
    }

    # Make sure $authorsForDropdown is made available the view
    return view('book.edit')->with([
        'book' => $book,
        'authorsForDropdown' => $authorsForDropdown
    ]);
```

Construct the dropdown (`<select>`) in the view using this data:

```html
<label for='author'>* Author:</label>
<select name='author' id='author'>
    <option value='' selected='selected' disabled='disabled'>Choose one...</option>
    @foreach($authorsForDropdown as $id => $name)
        <option value='{{ $id }}'>{{ $name }}</option>
    @endforeach
</select>
```

Update the `BookController@store` method so also save the author details about the book.

One way to do this:
```php
$author = Author::find($request->input('author'));
$book->author()->associate($author);
```

Or, just manually specify the `author_id` since we already have it in the request. Saves us a trip to the database to fetch the Author object.
```php
$book->author_id = $request->input('author');
```


## Custom model methods
We'll want to do the same thing on the *Edit* page.

Rather than duplicate the &ldquo;get authors for dropdown&rdquo; code, we can extract it out of the controller and add it as a method to the `Author` model:

```php
# app/Author.php
public static function getForDropdown()
{
    $authors = Author::orderBy('last_name')->get();
    foreach ($authors as $author) {
        $authorsForDropdown[$author->id] = $author->first_name.' '.$author->last_name;
    }
    return $authorsForDropdown;
}
```

Then in `BookController@create`:
```
$authorsForDropdown = Author::getForDropdown();
```

Other frequently used queries can also be abstracted, for example the following method could be added to the Book model:

```php
public static function getAllWithAuthors()
{
    return Book::with('author')->orderBy('id','desc')->get();
}
```


## Validation
```php
$this->validate($request, [
    'title' => 'required|min:3',
    'author' => 'required', # <----
    'published' => 'required|min:4|numeric',
    'purchase_link' => 'required|url',
    'cover' => 'required|url',
]);
```
