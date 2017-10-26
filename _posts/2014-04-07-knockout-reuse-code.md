---
layout: post
title: Knockout reuse code
tags: [knockout, ko, dry, reuse, extend, apply]
---

Here is simple example of knockout.js model code reuse:

Suppose we have some model:

```js
function Parent(name) {
    var self = this;
    self.name = ko.observable(name);
    self.message = ko.computed(function(){
        return 'Hello ' + self.name();
    });
    self.sayHi = function() {
        alert(self.message());
    }
}
```

And want reuse its members in another model, here is how to do it:

```js
function Child() {
    var self = this;
    Parent.apply(self, ['Alex']);
    //ko.utils.extend(self, new Parent('Alex')); // another way
    // From now there is all methods and properties of Parent model
    self.sayHi();
}
```

