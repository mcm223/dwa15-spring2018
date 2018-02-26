# Prep DigitalOcean for Laravel
With your Laravel application up and running locally, you now need to deploy it on your production server.

Before deploying, though, you need to do a few updates on your DigitalOcean server so it's set up with the necessary components that Laravel needs to run.

In these notes, we'll do the following:

1. Enable a swap file for more memory
2. Install Composer
3. Update PHP, install some necessary modules
3. Enable `mod_rewrite`

Tip: This note set includes some complex command line commands, each prefixed with a brief explanation of what the command does. To break down any one particular command to learn more, paste the command into <https://explainshell.com>.

All of the following commands are to be run on your DigitalOcean server, so the first thing you want to do is SSH into your DigitalOcean server (`ssh root@your.do.ip.address`), then proceed...

## Enable a swap file for more memory
Composer can be memory intensive, so we'll want to configure our low-memory DigitalOcean droplets to use a swap file.

>> "Swap is an area on a hard drive that has been designated as a place where the operating system can temporarily store data that it can no longer hold in RAM." -[ref](https://www.digitalocean.com/community/tutorials/how-to-add-swap-on-ubuntu-14-04)

To configure a swap file, run through the following commands.

First, create the swap file:

```xml
$ sudo fallocate -l 4G /swapfile
```

Next, adjust permissions on the resulting swap file so it isn't readable by anyone besides root:

```
$ sudo chmod 600 /swapfile
```

Next, tell the system to set up the swap space:

```xml
$ sudo mkswap /swapfile
```

Next, enable the swap space:

```
$ sudo swapon /swapfile
```

Finally, we want to make it so the server always enables this swap space, even after a reboot. To do this, open `/etc/fstab` with nano:

```
$ sudo nano /etc/fstab
```

...and paste in the following line at the end:

```
/swapfile   none    swap    sw    0   0
```

Save your changes (`ctrl` + `x`, then `y`, then *Enter*).

__Confirm it worked:__
You can confirm your swap file with the following command:

```
$ sudo swapon -s
```

Expected output:

```xml
Filename                Type        Size    Used    Priority
/swapfile               file        4194300 0       -1
```


## Install Composer
Move into your bin directory where you'll install Composer:

```bash
$ cd /usr/local/bin
```

Use cURL to download Composer and then execute the resulting download with PHP:

```bash
$ curl -sS https://getcomposer.org/installer | sudo php
```

Rename the composer executable to `composer` so it's convenient to invoke:

```bash
$ sudo mv composer.phar composer
```

Test it's working:

```bash
$ composer
```

See a list of Composer commands? Good, you're ready to move to the next step...


## Install/update necessary modules
The image we built our DigitalOcean Droplets from came installed with PHP 7.0 but Laravel requires >= 7.1.3. Thus, our next task is to bring PHP up to date, and we'll also install a few additional modules that Laravel needs.

To do this, we'll use `apt-get`, a command line utility for managing packages on Ubuntu systems (which our Droplets run on).

Run the following commands, one at a time. Follow the instructions to hit `Enter` or `Y` (yes) when prompted.

First, specify a new repository `apt-get` can download software packages from. The repository we're adding is [ppa:ondrej/php](https://launchpad.net/~ondrej/+archive/ubuntu/php), the primary source for PHP-related packages.


```bash
$ sudo add-apt-repository ppa:ondrej/php
```


Next, run `apt-get` to get the latest packages from your apt-get repositories (including your newly added one, `ppa:ondrej/php`)

```bash
$ sudo apt-get update
```

Now that everything is updated, we'll install `php7.2` as well as some other necessary modules:
```
$ sudo apt-get install php7.2 php7.2-mysql php7.2-xml php7.2-mbstring php7.2-xml zip unzip
```

Tell Apache to disable the older version of PHP:
```
$ sudo a2dismod php7.0
```

And enable PHP 7.2:
```
$ sudo a2enmod php7.2
```

Restart the Apache web server to make your changes take effect.
```
$ service apache2 restart
```

When you're done, you can confirm PHP command line is running the correct version:

```xml
$ php -v
PHP 7.2.2-3+ubuntu16.04.1+deb.sury.org+1 (cli) (built: Feb  6 2018 16:11:23) ( NTS )
Copyright (c) 1997-2018 The PHP Group
Zend Engine v3.2.0, Copyright (c) 1998-2018 Zend Technologies
    with Zend OPcache v7.2.2-3+ubuntu16.04.1+deb.sury.org+1, Copyright (c) 1999-2018, by Zend Technologies
```

You can also confirm Apache is running the correct version by visiting `http://your-digitalocean-ip/info.php` (assuming you have not deleted that `info.php` file.)


## Enable mod_rewrite
Laravel requires Apache's `mod_rewrite` for URL routing (a topic we'll cover in future lectures).

To enable this module, run the following command:

```xml
$ sudo a2enmod rewrite
```

Restart Apache to make these this change take effect:
```xml
$ service apache2 restart
```




## Server setup complete!
At this point, you DigitalOcean Droplet has everything it needs to run a Laravel app. You're ready to move on to the next steps of deploying.


## Reference
+ [Ubuntu: How to manage software repositories from the command line](https://help.ubuntu.com/community/Repositories/CommandLine)
+ [Discrepancy between PHP versions on local vs. DigitalOcean](https://gist.github.com/susanBuck/f949e701c239a7468de64cd89fe0347b)
