# Cookies and sessions

## Preface
Cookies and sessions are a fundamental part of how web sites work, so it's important to have some background info now, as we're covering all the essentials.

That being said, **your Project 2 does not need to use cookies or sessions.** We will use cookies and sessions later in the semester; right now, we just want a broad idea of how they work.

## Application memory
In a traditional stand-alone application, there's a memory system in place that is used to store and retrieve data as long as the application is running.

With web applications, our applications are effectively run every time the page loads, so we need some mechanism to store data between runs, i.e. between requests.

Example use case: remembering logged in users as they browse different pages on your application.

Retaining data between requests can be accomplished with cookies and sessions.

__Cookies__ are small text payloads made up of a key,value pair that are stored in a user's browser.
+ Servers can both set and retrieve cookies on a visitor's browser.
+ Cookies are domain specific&mdash; a server/site can only access cookies that it created.

__Sessions__ are also small text payloads made up of key,value pairs, but instead of being stored on the user's browser, they're stored on the site's server.

+ Sessions can be stored in a variety of ways on the server, such as in plain text files, or in a database.

Cookies and sessions are used, in combination, to recall information as a visitor uses your site.

Example:

Visitor logs in...
+ a *session* is created on the server; it contains the user's id (used to query all the info about that user from the database)
+ A *cookie* is also set that contains the name of the session that was just created

Visitor visits the site again...
+ PHP looks for the user cookie, and upon finding it, extracts the name of the session to look for
+ Session is loaded, and from it the user id is extracted and used to query the database for info about that user.


## Cookie example
For a more tangible example, we could use cookies to recall what the visitor's most recent search was.

### Setting cookies
Cookies can be set with PHP's [`setcookie`](http://php.net/manual/en/function.setcookie.php) function, e.g.:

```php
setcookie('recentSearch', $_POST['searchTerm']);
```

Because cookies are set as part of a request's header, you can not have *any* output to the page before setcookie is invoked.

E.g.

This would cause an error:
```php
echo 'hi!';
setcookie('recentSearch', $_POST['searchTerm']);
```

This would not
```php
setcookie('recentSearch', $_POST['searchTerm']);
echo 'hi!';
```


### Getting cookies
Cookies can be retrieved using PHP's [`$_COOKIE`](http://php.net/manual/en/reserved.variables.cookies.php) superglobal.

```php
dump($_COOKIE['recentSearch']);
```

Knowing this...


### Example
Update your `search.php` file with the following code:

```php
<?php

require('helpers.php');

if(isset($_COOKIE['recentSearch'])) {
    $recentSearch = $_COOKIE['recentSearch'];
}
else {
    $recentSearch = '';
}

setcookie('recentSearch', $_POST['searchTerm']);

dump('You recently searched for: '.$recentSearch);
dump('You just searched for: '.$_POST['searchTerm']);
```

Study the results by submitting your search form several times.
