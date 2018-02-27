# Directory structure
Most frequently used files/directories:

| Path  | Usage |
|---|---|
| `/routes/web.php`  | Where all your web **routes** go; think of this as the map of your application.  |
| `/app`  | Where all your **models** go (amongst other files/dirs that make up the core of your app)  |
| `/resources/views/` | Where all your **views** go |
| `/app/Http/Controllers` | Where all your **controllers** go |
| `/config/` | Where all your configurations are set |
| `/.env` | Where configurations that are specific to the *current* environment are set |
| `/composer.json` | Composer config file - Where you manage what dependencies your application has |

Learn more: [Laravel Docs: Directory Structure](https://laravel.com/docs/5.4/structure)

Remember that [PhpStorm's **Search Everywhere** tool can be invoked by hitting Shift twice](https://github.com/susanBuck/dwa15-spring2018/blob/master/misc/phpstorm.md#keymaps). This feature can help minimize the overwhelming nature of all the dirs/files in a framework like Laravel. 