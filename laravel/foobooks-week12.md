# Foobooks Week 12: Database interaction examples
The following is a rough outline of the modifications I'll make to Foobooks in Week 12's lecture.

__This should not be considered a stand-alone document; for full details please refer to the lecture video and the Foobooks code source.__

## View all books
- `BookController@index`
- `/resources/views/books/index.blade.php`

## View individual book
- `BookController@show`
- `/resources/views/books/show.blade.php`

## Add a book
Add the `old` helper for field values, and set default values to rapidly test:

+ Title: `Green Eggs & Ham`
+ Author: `Dr. Seuss`
+ Published Year: `1960`
+ Cover URL: `http://prodimage.images-bn.com/pimages/9780394800165_p0_v4_s192x300.jpg`
+ Purchase URL: `http://www.barnesandnoble.com/w/green-eggs-and-ham-dr-seuss/1100170349`

(We'll remove this when we're done debugging)


In `BookController.php` update `store` to save the book to the database:
```php
$book = new Book();
$book->title = $request->input('title');
$book->author = $request->input('author');
$book->published_year = $request->input('published_year');
$book->cover_url = $request->input('cover_url');
$book->purchase_url = $request->input('purchase_url');
$book->save();
```

Redirect to the book index and  [flash](https://laravel.com/docs/redirects#redirecting-with-flashed-session-data) a confirmation message which can be done via redirect's `with` method:
```
return redirect('/books')->with('alert', 'Your book was added.');
```

Then in `master.blade.php`:
```php
@if(session('message'))
    <div class='alert'>{{ session('message') }}</div>
@endif
```

Which can be styled however you want:
```css
.alert {
    background-color: yellow;
    width: 100%;
    position: fixed;
    top: 0;
    left: 0;
    padding: 5px;
    font-weight: bold;
    text-align: center;
}
```

## Edit a book
Start `books/edit.blade.php` by copying `books/create.blade.php`.

Create the routes to a) show the edit form and b) process the edit form:
```php
# Show the form to edit a specific book
Route::get('/books/{id}/edit', 'BookController@edit');

# Process the form to edit a specific book
Route::put('/books/{id}', 'BookController@update');
```

Note the use of the `put` method here. In order to submit via PUT we need to use [form method spoofing](https://laravel.com/docs/routing#form-method-spoofing) like so:

```php
<form method='POST' action='/books/{{ $book->id }}'>

    {{ method_field('put') }}

    {{ csrf_field() }}

    [...]
</form>
```

Edit method to show the form:
```php
/*
* GET /books/{id}/edit
*/
public function edit($id)
{
    $book = Book::find($id);

    if (!$book) {
        return redirect('/books')->with('alert', 'Book not found.');
    }

    return view('books.edit')->with([
        'book' => $book
    ]);
}
```

Update method to process the form:
```php
/*
* PUT /books/{id}
*/
public function update(Request $request, $id)
{
   $this->validate($request, [
       'title' => 'required',
       'published_year' => 'required|digits:4|numeric',
       'cover_url' => 'required|url',
       'purchase_url' => 'required|url',
   ]);

    $book = Book::find($id);

    $book->title = $request->input('title');
    $book->author = $request->input('author');
    $book->published_year = $request->input('published_year');
    $book->cover_url = $request->input('cover_url');
    $book->purchase_url = $request->input('purchase_url');
    $book->save();

    return redirect('/books/'.$id.'/edit')->with('alert', 'Your changes were saved.');
}
```


## Delete a book
Try this one on your own. I'll share an example solution next week.
