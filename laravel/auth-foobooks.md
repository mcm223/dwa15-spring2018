__Bonus material__

Integrating authentication into Project 4 is optional and this information will not be addressed in any of the progress log quick check questions.


## Integrating your existing data and authentication
Couple different approaches:

1. *One to Many*: Every book is connected to a single user
2. *Many to Many*: Individual books aren't associated with any one user. Instead users can favorite books, powered by a many to many relationship between books and users.

Let's go with #1.


## Connect books and users
To do this, we'll need a migration to update our `books` table to have the `user_id` foreign key. This is the same thing we did when connecting *authors* and *books*.

Create the migration:
```bash
$ php artisan make:migration connect_books_and_users
```

Fill in the migration:
```php
public function up()
{
    Schema::table('books', function (Blueprint $table) {

        # Add a new INT field called `user_id` that has to be unsigned (i.e. positive)
        $table->integer('user_id')->unsigned();

        # This field `user_id` is a foreign key that connects to the `id` field in the `authors` table
        $table->foreign('user_id')->references('id')->on('users');
    });
}

public function down()
{
    Schema::table('books', function (Blueprint $table) {

        # ref: http://laravel.com/docs/5.1/migrations#dropping-indexes
        $table->dropForeign('books_user_id_foreign');

        $table->dropColumn('user_id');
    });
}
```


## Update seeders
Because we've created a foreign key between `books` and `users`, make sure your UsersTableSeeder is invoked *before* the BooksTableSeeder:

```php
# DatabaseSeeder.php
$this->call(UsersTableSeeder::class);
$this->call(TagsTableSeeder::class);
$this->call(AuthorsTableSeeder::class);
$this->call(BooksTableSeeder::class);
$this->call(BookTagTableSeeder::class);
```

Finally, update the BooksTableSeeder so that each book is associated with a user. For our example, we'll associate every book to user id 1 (`jill@harvard.edu`).

```php
Book::insert([
    'created_at' => $timestampForThisBook,
    'updated_at' => $timestampForThisBook,
    'title' => $title,
    'author_id' => $author_id,
    'published' => $book['published'],
    'cover' => $book['cover'],
    'purchase_link' => $book['purchase_link'],
    'user_id' => 1, # <--- NEW LINE
]);
```



## Update models
Next, update the `Book` and `User` model so they're aware of this new relationship.

Add this to `Book.php`:
```php
public function user()
{
    return $this->belongsTo('App\User');
}
```

Add this to `User.php`:
```php
public function books()
{
    return $this->hasMany('App\Book');
}
```


## Your Books
Setup complete! Now let's make it so that when a user is logged in they only see *their* books.
Update the `index` method in the BookController like so:

```php
public function index()
{
    $user = $request->user();

    # Note: We're getting the user from the request, but you can also get it like this:
    //$user = Auth::user();
    
    if ($user) {
        # Approach 1)
        //$books = Book::where('user_id', '=', $user->id)->orderBy('title')->get();

        # Approach 2) Take advantage of Model relationships
        $books = $user->books()->orderBy('title')->get();

        # Get 3 most recently added books
        $newBooks = $books->sortByDesc('created_at')->take(3); # Query existing Collection
    } else {
        $books = [];
        $newBooks = [];
    }

    return view('book.index')->with([
        'books' => $books,
        'newBooks' => $newBooks,
    ]);
}
```


### View modifications
In addition to the above changes, we also made the modifications to the book index view:

+ Update the heading on the book index from *All Books* to *Your Books*.
+ Only show the new books call-out if there are new books.
+ If there are no books to show, link to the page to add a new book.

Study [/resources/book/index.blade.php](https://github.com/susanBuck/foobooks/blob/master/resources/views/book/index.blade.php) to see the details of these changes.

### Test it out
+ Login as Jill and you should see all the books (since they're all seeded to her)
+ Login as Jamal to see the &ldquo;blank slate&rdquo; page with no books to show.

<img src='http://making-the-internet.s3.amazonaws.com/laravel-books-and-no-books@2x.png' style='max-width:1070px;' alt='Comparing the view when you have books vs. when you do not'>




## Associating books with the logged in user
Update BookController's `store` method to set the book's user_id right before the book is saved:

```php
$book->user_id = $request->user()->id; # <--- NEW LINE
$book->save();
```

Once a book is created it is tied to that user; because of this there's no need to do anything with the *Edit Book* in regards to user associations.
