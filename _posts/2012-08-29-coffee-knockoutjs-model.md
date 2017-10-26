---
layout: post
title: Coffee knockoutJs model example
tags: [coffee, knockout, knockoutjs]
---

```coffee
class @Model
    constructor: ()->
        # protected member example
        hi = 'Hello, ' # protected

        # public observable member examples
        @firstName = ko.observable 'Alexandr'
        @lastName = ko.observable 'Marchenko'

        # public computed member example
        @fullName = ko.computed =>
            @firstName() + ' ' + @lastName()

        # another computed that uses protected memeber
        @greeting = ko.computed =>
            hi + @fullName()

    # public method (one per instance, can not be redefined via prototype)
    sayHello: => alert @greeting()

    # public method (one for all models, can be redefined like so Model.prototype.sayHi = function(){})
    sayHi: -> alert @greeting()

# exapmle how to apply bindings
$ -> ko.applyBindings new Model
```
