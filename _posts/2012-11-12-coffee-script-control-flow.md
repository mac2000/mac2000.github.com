---
layout: post
title: Coffee script - control flow
tags: [coffee, coffeescript, node, nodejs, async, parallel, wait, promise, asynchronous]
---

Many things in node.js runs asynchronously, so sooner or later you will be asking your self:

>> How to run multiple asynchronous commands and wait for all of them finish

You can find answer here: http://book.mixu.net/ch7.html
This is short note about principles and variations from article above.

Hello World
-----------

Lets asume that we have some array of items that must be processed asynchronously.

We have two functions: `async(arg, callback)` that will process one item per time and call callback with result when all asyn operations are done and `final(results)` function wich will be called once all items are processed.

    function async(arg, callback) {
        console.log('[>] ' + arg);
        var random_timeout = 1000 * (Math.floor(Math.random() * 3) + 1); // Used to simulate random execution time
        setTimeout(function () { // http.get, fs.readFile, etc...
            var result = arg * 2;
            console.log('[<] ' + arg);
            callback(result); // Do not forget to call callback(result) here
        }, random_timeout);
    }

    function final(results) { // Will be run once all jobs are done
        console.log('[+] DONE');
        console.log(results);
    }

Nothing very special here, but notice that in `async` function there will be `http.get`, `fs.readFile` or any other async operation instead of `setTimeout` that used for test purposes only. When you get your results just call callback function with them.

One by one
----------

In this case all jobs will be runned synchronously, and you will get result in same order as you give.

    function series(items, async, final) {
        var results = [];

        function launcher(item) {
            if (item) {
                async(item, function (result) {
                    results.push(result);
                    return launcher(items.shift());
                });
            } else {
                final(results);
            }
        }

        launcher(items.shift());
    }

Now if you will run this, you will get something like this:

    series([1, 2, 3, 4, 5, 6], async, final);

    [>] 1
    [<] 1
    [>] 2
    [<] 2
    [>] 3
    [<] 3
    [>] 4
    [<] 4
    [>] 5
    [<] 5
    [>] 6
    [<] 6
    [+] DONE
    [ 2, 4, 6, 8, 10, 12 ]

Full parallel
-------------

In this case all jobs will be run in parallel, but be carefull if you have too many jobs to run, this will be not best choice.

    function parallel(items, async, final) {
        var results = [], i;

        for (i = 0; i < items.length; i = i + 1) {
            async(items[i], function (result) {
                results.push(result);
                if (results.length === items.length) {
                    final(results);
                }
            });
        }
    }

Now if you will run this, you will get something like this:

    parallel([1, 2, 3, 4, 5, 6], async, final);

    [>] 1
    [>] 2
    [>] 3
    [>] 4
    [>] 5
    [>] 6
    [<] 1
    [<] 4
    [<] 3
    [<] 2
    [<] 5
    [<] 6
    [+] DONE
    [ 2, 8, 6, 4, 10, 12 ]

Notice that order of response are not same as order of given items, also notice that all six items are going to calculate immediately.

Limited parallel
----------------

The same as above, but with limit for parallel jobs to run

    function limited(items, async, final, limit) {
        var results = [],
            running = 0;

        function launcher() {
            while (running < limit && items.length > 0) {
                var item = items.shift();

                async(item, function (result) {
                    results.push(result);
                    running = running - 1;

                    if (items.length > 0) {
                        launcher();
                    } else if (running === 0) {
                        final(results);
                    }
                });

                running = running + 1;
            }
        }

        launcher();
    }

Now if you will run this, you will get something like this:

    limited([1, 2, 3, 4, 5, 6], async, final, 2);

    [>] 1
    [>] 2
    [<] 2
    [>] 3
    [<] 1
    [>] 4
    [<] 3
    [>] 5
    [<] 4
    [>] 6
    [<] 5
    [<] 6
    [+] DONE
    [ 4, 2, 6, 8, 10, 12 ]

Notice that there is also another order or response item, but now you can control how much jobs will be run concurently.

You can use:

    var os = require('os');
    var limit = os.cpus().length;

to limit number of jobs.

All together
------------

**control_flow.js**

    // http://caolanmcmahon.com/posts/writing_for_node_and_the_browser/
    (function (exports) {
        "use strict";

        exports.series = function (items, async, final) {

            var results = [];

            function launcher(item) {
                if (item) {
                    async(item, function (result) {
                        results.push(result);
                        return launcher(items.shift());
                    });
                } else {
                    final(results);
                }
            }

            launcher(items.shift());
        };

        exports.parallel = function (items, async, final) {
            var results = [], i;

            for (i = 0; i < items.length; i = i + 1) {
                async(items[i], function (result) {
                    results.push(result);
                    if (results.length === items.length) {
                        final(results);
                    }
                });
            }
        };

        exports.limited = function (items, async, final, limit) {
            var results = [],
                running = 0;

            function launcher() {
                while (running < limit && items.length > 0) {
                    var item = items.shift();

                    async(item, function (result) {
                        results.push(result);
                        running = running - 1;

                        if (items.length > 0) {
                            launcher();
                        } else if (running === 0) {
                            final(results);
                        }
                    });

                    running = running + 1;
                }
            }

            launcher();
        };

    })(typeof exports === 'undefined' ? this.control_flow = {} : exports);

**test.js**

    var control_flow = require('./control_flow');

    function async(arg, callback) {
        console.log('[>] ' + arg);
        var random_timeout = 1000 * (Math.floor(Math.random() * 3) + 1);
        setTimeout(function () {
            var result = arg * 2;
            console.log('[<] ' + arg);
            callback(result);
        }, random_timeout);
    }

    function final(results) {
        "use strict";
        console.log('[+] DONE');
        console.log(results);
    }

    //control_flow.series([1, 2, 3, 4, 5, 6], async, final);
    //control_flow.parallel([1, 2, 3, 4, 5, 6], async, final);
    control_flow.limited([1, 2, 3, 4, 5, 6], async, final, 2);

So now we can just require our helper and run it as we need, but what about coffee?

**control_flow.coffee**

    # http://www.plexical.com/blog/2012/01/25/writing-coffeescript-for-browser-and-nod/
    control_flow = exports? and @ or @control_flow = {}

    control_flow.series = (items, async, final) ->
        results = []

        launcher = (item) ->
            if item
                async(item, (result) ->
                    results.push result
                    return launcher(items.shift())
                )
            else
                final(results)

        launcher(items.shift())

    control_flow.parallel = (items, async, final) ->
        results = []

        items.forEach (item) ->
            async(item, (result) ->
                results.push result
                if results.length == items.length
                    final(results)
            )

    control_flow.limited = (items, async, final, limit = 2) ->
        results = []
        running = 0

        launcher = () ->
            while running < limit and items.length > 0
                item = items.shift()

                async(item, (result) ->
                    results.push result
                    running--
                    if items.length > 0
                        launcher()
                    else if running is 0
                        final(results)
                )
                running++

        launcher()

**test.coffee**

    control_flow = require './control_flow'

    async = (arg, callback) ->
        console.log "[>] #{arg}"
        random_timeout = 1000 * Math.floor(Math.random() * 3) + 1
        setTimeout ( ->
            console.log "[<] #{arg}"
            result = arg * 2
            callback(result)
        ), random_timeout

    final = (results) ->
        console.log "[+] DONE"
        console.log results

    # control_flow.series [1, 2, 3, 4, 5, 6], async, final
    # control_flow.parallel [1, 2, 3, 4, 5, 6], async, final
    control_flow.limited [1, 2, 3, 4, 5, 6], async, final

Again thanks to authors of http://book.mixu.net/ch7.html page for such great info. Also notice that at moment there is many ready to use libraries that allow do all this.

ES6 Promise Serial
------------------

    var series = (items, async) => new Promise(resolve => {
      let results = []

      const launcher = item => {
        if (item) {
          async(item).then(result => {
            results.push(result)
            return launcher(items.shift())
          })
        } else {
          resolve(results)
        }
      }

      launcher(items.shift())
    })

    var urls = ['https://angular-university.io/api/courses/getting-started-with-angular2', 'https://angular-university.io/api/courses/angular2-http']
    var async = url => fetch(url).then(r => r.json())

    series(urls, async).then(i => console.log(i))
