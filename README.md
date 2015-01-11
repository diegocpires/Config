# Respect\Config

[![Build Status](https://img.shields.io/travis/Respect/Config.svg?style=flat-square)](http://travis-ci.org/Respect/Config)
[![Code Coverage](https://img.shields.io/scrutinizer/coverage/g/Respect/Config.svg?style=flat-square)](https://scrutinizer-ci.com/g/Respect/Config/?branch=master)
[![Latest Stable Version](https://img.shields.io/packagist/v/respect/config.svg?style=flat-square)](https://packagist.org/packages/respect/config)
[![Total Downloads](https://img.shields.io/packagist/dt/respect/config.svg?style=flat-square)](https://packagist.org/packages/respect/config)
[![License](https://img.shields.io/packagist/l/respect/config.svg?style=flat-square)](https://packagist.org/packages/respect/config)

A powerful, small, deadly simple configurator and dependency injection container made to be easy. Featuring:

* INI configuration files only. Simpler than YAML, XML or JSON (see samples below).
* Uses the same native, fast parser that powers php.ini.
* Extends the INI configuration with a custom dialect.
* Implements lazy loading for object instances.

## Installation

The package is available on [Packagist](https://packagist.org/packages/arara/process).
You can install it using [Composer](http://getcomposer.org).

```bash
composer require respect/config
```

Works on PHP 5.3 and 5.4 only.

## Autoloading

You can set up Respect\Config for autoloading. We recommend using the
SplClassLoader. Here's a nice sample:

````php
set_include_path('/my/library' . PATH_SEPARATOR . get_include_path());
require_once 'SplClassLoader.php';
$respectLoader = new \SplClassLoader();
$respectLoader->register();
````


## Running Tests

We didn't created our tests just for us to apreciate. To run them,
you'll need phpunit 3.5 or greater. Then, just chdir into the `/tests` folder
we distribute and run them like this:

````bash
cd /my/RespectConfig/tests
phpunit .
````

You can tweak the phpunit.xml under that `/tests` folder to your needs.
#
=============

## Variable Expanding

myconfig.ini:

````ini
db_driver = "mysql"
db_host   = "localhost"
db_name   = "my_database"
db_dsn    = "[db_driver]:host=[db_host];dbname=[db_name]"
````

myapp.php:

````php
$c = new Container('myconfig.ini');
echo $c->db_dsn; //mysql:host=localhost;dbname=my_database
````

Note that this works only for variables without ini [sections].

## Sequences

myconfig.ini:

````ini
allowed_users = [foo,bar,baz]
````

myapp.php:

````php
$c = new Container('myconfig.ini');
print_r($c->allowed_users); //array('foo', 'bar', 'baz')
````

Variable expanding also works on sequences. You can express something like this:

myconfig.ini:

````ini
admin_user = foo
allowed_users = [[admin_user],bar,baz]
````

myapp.php:

````php
$c = new Container('myconfig.ini');
print_r($c->allowed_users); //array('foo', 'bar', 'baz')
````

## Constant Evaluation

myconfig.ini:

````ini
error_mode = PDO::ERRMODE_EXCEPTION
````

Needless to say that this would work on sequences too.

## Instances

Using sections

myconfig.ini:

````ini
[something stdClass]
````

myapp.php:

````php
$c = new Container('myconfig.ini');
echo get_class($c->something); //stdClass
````

Using names

myconfig.ini:

````ini
date DateTime = now
````

myapp.php:

````php
$c = new Container('myconfig.ini');
echo get_class($c->something); //DateTime
````

## Callbacks

myconfig.ini:

````ini
db_driver = "mysql"
db_host   = "localhost"
db_name   = "my_database"
db_user   = "my_user"
db_pass   = "my_pass"
db_dsn    = "[db_driver]:host=[db_host];dbname=[db_name]"
````


myapp.php:

````php
$c = new Container('myconfig.ini');
$c->connection = function() use($c) {
    return new PDO($c->db_dsn, $c->db_user, $c->db_pass);
};
echo get_class($c->connection); //PDO
````

## Instance Passing

myconfig.ini:

````ini
[myClass DateTime]

[anotherClass stdClass]
myProperty = [myClass]
````

myapp.php:

````php
$c = new Container('myconfig.ini');
echo get_class($c->myClass); //DateTime
echo get_class($c->anotherClass); //stdClass
echo get_class($c->myClass->myProperty); //DateTime
````

Obviously, this works on sequences too.

## Instance Constructor Parameters

Parameter names by reflection:

myconfig.ini:

````ini
[connection PDO]
dsn      = "mysql:host=localhost;dbname=my_database"
username = "my_user"
password = "my_pass"
````

Method call by sequence:

myconfig.ini:

````ini
[connection PDO]
__construct = ["mysql:host=localhost;dbname=my_database", "my_user", "my_pass"]
````

Using Names and Sequences:

myconfig.ini:

````ini
connection PDO = ["mysql:host=localhost;dbname=my_database", "my_user", "my_pass"]
````

## Instantiation by Static Factory Methods

myconfig.ini:

````ini
[y2k DateTime]
createFromFormat[] = [Y-m-d H:i:s, 2000-01-01 00:00:01]
````

## Instance Method Calls

myconfig.ini:

````ini
[connection PDO]
dsn             = "mysql:host=localhost;dbname=my_database"
username        = "my_user"
password        = "my_pass"
setAttribute    = [PDO::ATTR_ERRMODE, PDO::ATTR_EXCEPTION]
exec[]          = "SET NAMES UTF-8"
````

## Instance Properties

myconfig.ini:

````ini
[something stdClass]
foo = "bar"
````

# Known Limitations

* Variable expanding only works for unsectioned keys.
* Empty strings, zeros and null are not properly treated yet.
* Constructors with non-null default parameter values may not work properly yet.
* The only way to use magic methods is to call them explicitly with __call
* Circular references haven't been tested and may not work
* May not work properly with the following conditions:
    * Static and normal methods with the same names within the same class
    * Methods and properties with same names within the same class
    * Methods and properties with same names as constructor parameters

Luckly, most of these limitations are known to be PHP bad practices. Keep up the
good work and you'll never face them.

# License Information

See [LICENSE](LICENSE) file.
