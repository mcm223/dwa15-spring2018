# Production database setup

With the topic of __migrations__ and __seeding__ behind us, the next logical step is to set up the production database.

Here is a summary of the steps we'll take (all on the production server) to get this done:

1. Make sure your production code is up to date.
2. Find your default MySQL password.
3. Create a new database.
4. Set up these credentials in your production `.env` file
5. Run migrations (to build tables) and seeders (to fill tables).




## 1. Make sure your production code is up to date
Before proceeding with the next steps, the first thing you want to do is make sure any database related changes in your local code base have been added, committed, and pushed to Github, then pulled down to your production server.

Specifically, you want to make sure any __migration files__ and any __seeder files__ are in existence and up to date on your production server. The last step in this document will be to run these migrations and seeders, so we want to make sure they're ready to go.

So, do a `git status` locally. Are there any new or changed migration or seeder files? If so, go ahead and sync up so your live server also has these changes.

Also recall that you want to run `composer install --no-dev` on production when deploying not only to ensure all packages are up to date, but to make sure your Composer autoloading class map file reflects any new classes (including seeds/migrations) that have been added.

## 2. Find your default MySQL password

When you first SSH into your DigitalOcean server, you should see a welcome message (a.k.a &ldquo;[message of the day](http://en.wikipedia.org/wiki/Motd_(Unix))&rdquo;).

This message includes a note indicating that your default MySQL password is saved in `/root/.digitalocean_password`

<img src='http://making-the-internet.s3.amazonaws.com/laravel-motd-mysql-password@2x.png' style='max-width:923px;' alt=''>

Thus, if you use `cat` to view this file, you can see your password:
```bash
root@Fall2016:~# cat /root/.digitalocean_password
root_mysql_pass="9f15faddf3d4c9825fac6f22ab042ad4b47919ff063bd063"
```

In this example, the password is `9f15faddf3d4c9825fac6f22ab042ad4b47919ff063bd063`


## 3. Create a new database
When working locally, we create new databases using phpMyAdmin. phpMyAdmin is not installed on your DigitalOcean server by default, so we'll instead use the MySQL command line tool to create a new database.

While SSH'd into your DigitalOcean server, run the following command to start the MySQL command line tool:

```bash
$ mysql -u root -p
```

When it prompts for your password, use the MySQL password you copied in the previous step.

If all goes well, you should see some info about MySQL, followed by the `mysql> ` prompt.

<img src='http://making-the-internet.s3.amazonaws.com/laravel-mysql-tool-on-digital-ocean@2x.png' style='max-width:617px; width:100%' alt=''>



Run the SQL command `SHOW DATABASES;` to see what databases exist on your server (there should be 3 default ones).

(Note: Every SQL command you enter should end with a semi-colon)

```xml
mysql> SHOW DATABASES;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| mysql              |
| performance_schema |
| sys                | 
+--------------------+
3 rows in set (0.00 sec)
```

Next, you can create a new database using the SQL command `CREATE DATABASE <db-name>;`.

For example:
```xml
mysql> CREATE DATABASE foobooks;
Query OK, 1 row affected (0.00 sec)
```

You can run `SHOW DATABASES;` again to confirm the new database exists.

When done, run `exit;` to get out of the MySQL command line tool.




## 4. Set up these credentials in your production `.env` file
In order for your application to interact with this database, you have to update your production `.env` file with the correct database credentials.

While SSHd into your DigitalOcean server, open your project's `.env` file in nano.

For example:

```bash
$ cd /var/www/html/foobooks/
$ nano .env
```

In your .env file add the following configs:

```xml
DB_DATABASE=foobooks
DB_USERNAME=root
DB_PASSWORD=[your mysql password goes here]
```

When you're done updating your values, save your changes and exit nano (`ctrl + x`, then `y`, then *Enter*).

Next, run `php artisan config:clear` to ensure Laravel detects these changes to your configs.

With the above steps complete, you next want to confirm these credentials work.

If you still have the [/debug route](https://github.com/susanBuck/dwa15-fall2017/blob/master/03_Laravel/24_Local_database_config.md#test-your-connection) from when you were configuring your local database connection, you can use that again to confirm your live database connection.




## 5. Run migrations (to build tables) and seeders (to fill tables)
The final step is to run your migrations on your live server to build your tables, then run your seeders to fill these tables with some sample data.

First, the migrations:
```xml
$ php artisan migrate
```

When you run this command, it will show you a warning, `Application In Production!` and ask you to confirm you wish to run the migration. Type `y` and hit `Enter` to confirm that you do.

<img src='http://making-the-internet.s3.amazonaws.com/laravel-run-migrations-on-digital-ocean@2x.png' style='max-width:771px; width:100%' alt='Run migrations on DigitalOcean'>

Assuming your migrations successfully ran, you can now run your seeders:

```xml
$ php artisan db:seed
```

<img src='http://making-the-internet.s3.amazonaws.com/laravel-run-seeders-on-digital-ocean@2x.png' style='max-width:757px; width:100%' alt='Running seeders on DigitalOcean'>




## Viewing your data
When working locally, to confirm your tables were built and data was entered, you loaded phpMyAdmin to open the `foobooks` database and examine the tables.

As mentioned above, though, we don't have phpMyAdmin on the DigitalOcean server. As a substitute, we'll use the MySQL command line tool again. The following commands should be all you need to cover the basics of examining tables.


Start the MySQL command line tool:
```xml
mysql -u root -p
```

Show all the databases that exist on the server:
```xml
SHOW DATABASES;
```

"Open" a particular database:
```xml
USE foobooks;
```

Show all the tables in a opened database:
```xml
SHOW TABLES;
```

Show all the rows in a table:
```
SELECT * FROM books;
```

Show all the rows in a table in a vertical fashion (can more be readable than the default horizontal table layout):
```
SELECT * FROM books\G;
```

If something goes wrong with your migrations or seeders and you want to &ldquo;start over&rdquo;, you could delete individual tables...

```
DROP TABLE migrations;
DROP TABLE books;
[...etc...]
```

Or, to delete all the tables at once, simply delete the entire database and recreate it:

```
DROP DATABASE foobooks;
CREATE DATABASE foobooks;
```


## Troubleshooting

### Error: `Access denied`
If you try and run your migrations but they fail, it may be because the DB credentials you put into `.env` are incorrect. If that happens, you may see a message similar to this:

```xml
SQLSTATE[28000] [1045] Access denied for user 'root'@'localhost' (using password: YES)
```

If this happen, double check your credentials and try again.


### Error: `Class BooksTableSeeder does not exist`
If you see a message telling you your seeder files does not exist, for example...

```xml
Class BooksTableSeeder does not exist
```

...try running `composer dump-autoload`. This will refresh the class list the the composer autoloader has, which can be a needed step when new classes are added.




## Tips


### Changing your default MySQL password
The following command can be used to change the default MySQL password for `root`:

```sql
mysqladmin -u root -p'currentpassword' password newpassword
```

(the lack of a space before `'currentpassword'` is intentional)

Note:
+ If you change your password, remember to update what's in your `.env` file.
+ Changing your password will not update `/root/.digitalocean_password`

### mysql_secure_installation
When you SSH into DigitalOcean, you may have noticed a message prompting you to run the command `mysql_secure_installation`:

<img src='http://making-the-internet.s3.amazonaws.com/laravel-encouraged-to-run-mysql-secure@2x.png' class='' style='max-width:827px; width:100%' alt='mysql_secure_installation'>

When you run the suggest command, `mysql_secure_installation`, you'll be prompted to make some setting changes that will make MySQL more secure.

[See the process here...](http://making-the-internet.s3.amazonaws.com/laravel-mysql-secure-installation@2x.png)

[You can read more about mysql_secure_installation here...](http://dev.mysql.com/doc/refman/5.0/en/mysql-secure-installation.html)


### Creating new MySQL users
In the instructions in these notes, we used a default MySQL user called `root`, which has full access to any database we have running in MySQL.

If you are running multiple applications/databases on the same server, it's more secure to create new MySQL users for each application/database. Those MySQL users should only have permissions to work with *their* given database. This way, if a hacker compromises the credentials for one of the MySQL users, that hacker won't have access to your other databases.

In the interest of time, and to reduce our room for error, we're not following this practice in this course. In a real world application, however, you should consider implementing this security best practice.

If you want to learn more you can read this article: [DigitalOcean: How To Create a New User and Grant Permissions in MySQL](https://www.digitalocean.com/community/tutorials/how-to-create-a-new-user-and-grant-permissions-in-mysql).
