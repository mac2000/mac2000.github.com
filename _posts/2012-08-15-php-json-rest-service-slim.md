---
layout: post
title: Php rest json service via slim
tags: [rest, restfull, json, jsonp, service, slim]
---

http://mac-blog.org.ua/images/slim/slim_list.png

http://mac-blog.org.ua/images/slim/slim_auth.png

Main idea of project is to create reusable peaces of restfull json(p) service.

Slim framework is used as base and have ability to process requests with any method and optional arguments, so our code will catch all calls like this one: `/api/(:entity)(/:id)`, where `entity` - is handler class name wich must implement our specific interface, and `id` is optional argument for update and delete.

Here is how it will be looks like:

```php
$app = new Slim();
$app->map('/api/:entity(/:id)', function($entity, $id = null){
    //TODO: process request
})->via('GET', 'POST', 'PUT', 'DELETE')->conditions(array('id' => '\d+'));
$app->run();
```

So now all is left is to check that `entity` is declared class that implements our interface and call appropriate method from it, by the way here is interface:

```php
abstract class RestApiInterface extends Singleton {
    protected $db;
    protected $app;
    protected function __construct() {
        $this->db = Database::getInstance();
        $this->app = Slim::getInstance();
    }
    abstract public function all();
    abstract public function one($id);
    abstract public function add($data);
    abstract public function put($id, $data);
    abstract public function del($id);
}
```

So each `entity` must provide basic CRUD operations plus one for validation.

All this is back end, for front end we will create simple javascript helper:

```js
/**
 * Usage examples:
 * var users = $.restInterfaceTo('/slim/api/users');
 * users.all(console.log);
 * users.one(1, console.log);
 * users.add({name: 'Alex', mail: 'alex@example.com', age: 27}, console.log);
 * users.put(3, {name: 'alex', mail: 'alex@gmail.com', age: 23}, console.log);
 * users.del(3, console.log);
 */
;(function(window, document, $, undefined){
    $.extend({
        restInterfaceTo: function(url) {
            if(!url) $.error('URL is required');

            return {
                url: url,
                all: function(success, error) {
                    $.ajax({
                        type: 'GET',
                        url: url,
                        dataType: 'json',
                        success: success,
                        error: error
                    });
                },
                one: function(id, success, error) {
                    $.ajax({
                        type: 'GET',
                        url: url + '/' + id,
                        dataType: 'json',
                        success: success,
                        error: error
                    });
                },
                add: function(data, success, error) {
                    $.ajax({
                        type: 'POST',
                        contentType: 'application/json',
                        url: url,
                        dataType: 'json',
                        data: JSON.stringify(data),
                        success: success,
                        error: error
                    });
                },
                put: function(id, data, success, error) {
                    $.ajax({
                        type: 'PUT',
                        contentType: 'application/json',
                        url: url + '/' + id,
                        dataType: 'json',
                        data: JSON.stringify(data),
                        success: success,
                        error: error
                    });
                },
                del: function(id, success, error) {
                    $.ajax({
                        type: 'DELETE',
                        url: url + '/' + id,
                        dataType: 'json',
                        success: success,
                        error: error
                    });
                }
            };
        }
    });
})(window, document, jQuery);
```

Source codes:

https://github.com/mac2000/slim-json-rest-service-example
