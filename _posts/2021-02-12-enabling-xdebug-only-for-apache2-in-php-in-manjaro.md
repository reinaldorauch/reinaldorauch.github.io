---
layout: post
title: "Enabling XDebug for apache2 in php in Manjaro"
---

# Enabling XDebug only for apache2 in php in Manjaro #

So, when you're developing PHP locally normally you want a development environment so that you can easly debug your app. The _de-facto_ extension to do so in PHP is XDebug. In Manjaro, the instalation of XDebug is simple:

```bash
$ sudo pacman -S php-debug
```

It will create a file in the conf.d directory in the php's configuration directory, in `/etc/php/conf.d/xdebug.ini` to initialize it. But this comes with a caveat: Unlike Debian that splits the configuration between the environments that php can run (apache2, cli or php-fpm), in Arch and therefore Manjaro, it enables it globally. We don't want to be that way always if you don't develop for the CLI.

So, how you can only enable xdebug when running php from the apache2 webserver?

The way I found to do it is leveraging the ability of setting php settings within the apache2's php extension with `php_admin_value` directive in apache2 config files.

First, we need to remove or change the name of the `/etc/php/conf.d/xdebug.ini` so xdebug doesn't aways load:

```bash
# backing-up it so we can restore it later if we want
$ sudo mv /etc/php/conf.d/xdebug.ini /etc/php/conf.d/xdebug.ini.bak 
```

Then, now we can insert these configurations in apache2. Opening the `/etc/httpd/conf/extra/php_module.conf` file we will see something like this: (it comes with the libapache2-php-module package)

```ApacheConf
# Required modules: dir_module, php_module

<IfModule dir_module>
  <IfModule php_module>
    DirectoryIndex index.php index.html
    <FilesMatch "\.php$">
      SetHandler application/x-httpd-php
    </FilesMatch>
    <FilesMatch "\.phps$">
      SetHandler application/x-httpd-php-source
    </FilesMatch>
  </IfModule>
</IfModule>
```

It says to apache how to handle .php and .phps files if the php_module is loaded. We can use this conditional to insert the xdebug configuration to load the extension and effectivelly only load it here.
Just add these lines before the closing of the inner IfModule directive:

as such:

```ApacheConf
# Required modules: dir_module, php_module

<IfModule dir_module>
  <IfModule php_module>
    DirectoryIndex index.php index.html
    <FilesMatch "\.php$">
      SetHandler application/x-httpd-php
    </FilesMatch>
    <FilesMatch "\.phps$">
      SetHandler application/x-httpd-php-source
    </FilesMatch>

    # these ones below

    # here it loads the extension as if it was in 
    # /etc/php/conf.d/xdebug.ini file
    php_admin_value zend_extension xdebug.so

    # Setting the debug mode
    php_admin_value xdebug.mode debug

    # Saying to xdebug start with every request
    php_admin_value xdebug.start_with_request yes

    # Saying to xdebug what tcp port to listen for a 
    # client to connect
    php_admin_value xdebug.client_port 9000
  </IfModule>
</IfModule>
```

I choose these configs to make xdebug work with the xdebug vscode extension, but you can pass any config xdebug expects. You can see all the configs in this [link](https://xdebug.org/docs/all_settings).

If we done all correctly, restarting the httpd server should do the trick:

```bash
$ sudo systemctl restart httpd
```

A quick check with in a `phpinfo()` call in any page youre working on you will able to see if the extension was loaded.

And to know if the extension didn't load in cli you could use the command

```bash
$ php -m | grep xdebug
# nada
```

An empty result is what you expect.

There it is!

Thanks for reading and have a good one. Stay safe!

Reinaldo A. C. Rauch