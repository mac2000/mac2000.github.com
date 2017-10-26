---
layout: post
title: UML basics
tags: [uml]
---

Here is simples ever UML basics

Association
===========


Unidirectional
--------------

One object store other

![Unidirectional Association](/examples/uml_basics/Association/Unidirectional/UML.svg)

```php
class Engine {}

class Car {
    /**
    * @var Engine
    */
    protected $engine;
}
```

Bidirectional
-------------

Both objects know about each other

![Bidirectional Association](/examples/uml_basics/Association/Bidirectional/UML.svg)

```php
class Book {
    /**
    * @var Person
    */
    protected $person;
}

class Person {
    /**
    * @var array Books[]
    */
    protected $books;
}
```


Composition
-----------

Coupling (dependency) - object not only knowing about its dependency but also creates it. Todays trend is Dependency Injection which expects that all that objects will be passed to constructors or methods (Dependency Association)

![Composition Association](/examples/uml_basics/Association/Composition/UML.svg)

```php
class Set {}

class Slim {
    /**
    * @var Set
    */
    protected $container;

    function __construct() {
        $this->container = new Set();
    }

    // -- OR --

    /**
    * @return Set
    */
    public function getContainer() {
        if ($this->container == null) {
            $this->container = new Set();
        }
        return $this->container;
    }
}
```


Aggregation
-----------

Object knows about its children but does not create them, it recieves them through methods and/or constructor

![Aggregation Association](/examples/uml_basics/Association/Aggregation/UML.svg)

```php
class Swift_Transport {}

class Swift_Mailer {
    /**
    * @var Swift_Transport
    */
    protected $transport;

    /**
    * @param Swift_Transport $transport
    */
    public function __construct(Swift_Transport $transport)
    {
        $this->transport = $transport;
    }

    // -- OR --

    /**
    * @param Swift_Transport $transport ;
    * @return Swift_Mailer
    */
    public static function newInstance(Swift_Transport $transport)
    {
        return new self($transport);
    }
}
```


Dependency
==========

One object uses another, but not store it

![Dependency](/examples/uml_basics/Dependency/UML.svg)

```php
class Request {}

class HomeController {
    public function indexAction(Request $request) {}
}
```


Realization (Implementation)
============================

![Implementation](/examples/uml_basics/Realization/UML.svg)

```php
interface LoggerInterface {}

class ConsoleLogger implements LoggerInterface {}
```


Generalization (Inheritance)
============================

![Inheritance](/examples/uml_basics/Generalization/UML.svg)

```php
class Response {}

class HttpResponse extends Response {}
```
