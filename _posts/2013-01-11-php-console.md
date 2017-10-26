---
title: PHP console
layout: post
keywords: [php, console]
---

Python has great feature, you can create file like this one:

test.py
-------

```python
#!/usr/bin/env python
#-*- coding: utf-8 -*-

def test():
    print 'Hello World';

if __name__ == "__main__":
    test()
```

And use it right from console or import it to another script.

Now lets look how to acomplish the same behaviour in php.

Found at: http://stackoverflow.com/questions/2413991/php-equivalent-of-pythons-name-main

test.php
--------

```php
<?php
function test() {
    echo 'hello world'.PHP_EOL;
}

if(!debug_backtrace()) {
    test();
}
```
