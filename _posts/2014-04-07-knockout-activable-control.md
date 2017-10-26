---
layout: post
title: Knockout activable control
tags: [knockout, ko, dry, active, click, document]
---

I need some kind of "activable" control. When I click on trigger node, it must receive "active" class, so its children become visible. But on click on elements outside my control - it should become inactive.

Here is full example:

```html
<!doctype html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Activable Example</title>
    <style>
        .activable {
            display: none;
        }

        .active > p {
            color: red;
        }

        .active > .activable {
            display: block;
        }
    </style>
</head>
<body>

    <div data-bind="activable">
        <p>First</p>
        <div class="activable">
            Lorem ipsum dolor sit amet, consectetur adipisicing elit. Praesentium, a, ipsum iste labore libero reprehenderit illo et consequuntur quaerat dolorem alias ea adipisci dignissimos accusantium illum at architecto eius officia?

            <div data-bind="activable">
                <p>Third</p>
                <div class="activable">Lorem ipsum dolor sit amet, consectetur adipisicing elit. Praesentium, a, ipsum iste labore libero reprehenderit illo et consequuntur quaerat dolorem alias ea adipisci dignissimos accusantium illum at architecto eius officia?</div>
            </div>
        </div>
    </div>
    <div data-bind="activable: 'active'">
        <p>Second</p>
        <div class="activable">Lorem ipsum dolor sit amet, consectetur adipisicing elit. Praesentium, a, ipsum iste labore libero reprehenderit illo et consequuntur quaerat dolorem alias ea adipisci dignissimos accusantium illum at architecto eius officia?</div>
    </div>


    <script src="http://cdnjs.cloudflare.com/ajax/libs/knockout/3.1.0/knockout-min.js"></script>
    <script>
        ko.bindingHandlers.activable = {
            init: function(element, valueAccessor, allBindings, viewModel, bindingContext) {
                var activeCls = valueAccessor() || 'active';

                function isDescendant(parent, child) {
                    var node = child.parentNode;
                    while (node != null) {
                        if (node == parent) {
                            return true;
                        }
                        node = node.parentNode;
                        }
                    return false;
                }

                function activate(e) {
                    ko.utils.toggleDomNodeCssClass(element, activeCls, true);
                }

                function deactivate(e) {
                    if (!isDescendant(element, e.target || e.srcElement)) {
                        e.stopPropagation();
                        ko.utils.toggleDomNodeCssClass(element, activeCls, false);
                    }
                }

                ko.utils.registerEventHandler(element, 'click', activate);
                ko.utils.registerEventHandler(element, 'touchend', activate);
                ko.utils.registerEventHandler(document, 'click', deactivate);
                ko.utils.registerEventHandler(document, 'touchend', deactivate);
            },
            update: function(element, valueAccessor, allBindings, viewModel, bindingContext) {}
        };
    </script>
    <script>
        function Model() {
            var self = this;
        }
        ko.applyBindings(new Model());
    </script>
</body>
</html>
```
