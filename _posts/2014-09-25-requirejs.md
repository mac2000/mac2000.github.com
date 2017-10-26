---
layout: post
title: RequireJS how to require modules on the fly
tags: [rjs, requirejs, amd]
---

Short note about RequireJS.

We have simple form which have one text field with auto suggest and button.

Usually to make this work we have to include all scripts into page, but what if user never wants to use this form. RequireJS makes it possible to require needed scripts on the fly.


**index.html**

```html
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>Require.JS Sample Project</title>
</head>
<body>
	<input type="search">

	<button>Say HI</button>

	<script src="js/require.js" data-main="js/index"></script>
</body>
</html>
```


**js/index.js**

```js
require(['domReady!'], function(document) {
	var input = document.querySelector('input');

	input.addEventListener('input', function(event){
		var self = this;
		require(['modules/suggest'], function(suggest){
			suggest.byTerm(self.value);
		});
	});

	document.querySelector('button').addEventListener('click', function(event){
		event.preventDefault();

		require(['modules/greeting'], function(greeting){
			greeting.sayHi(input.value);
		});
	});
});
```

Notice how we require modules only when we need them.

So when user inputs something we require suggest module (this happens only once), when user clicks on button we require another module.


**js/mobules/suggest.js**

```js
define(['modules/ajax'], function(ajax){
	return {
		byTerm: function(term){
			ajax.getJSON('http://example.com?temr=' + term, function(response){
				console.log(response);
			});
		}
	};
});
```

Here is suggest module realization, notice that for it to work we need ajax module which is loaded dynamically.


**js/modules/ajax.js**

```js
define(function(){
	return {
		getJSON: function(url, callback){
			console.log('Retrieving JSON from ' + url);
			callback({success: true, message: 'ok'});
		}
	};
});
```

**js/modules/greeting.js**

```js
define(function(){
	return {
		sayHi: function(name){
			alert('Hello ' + name);
		}
	};
});
```

Greeting module is really simple and do nothing here.


Bundling and minifying
----------------------

```sh
r.js.cmd -o build.js
```

**build.js**

```js
({
	preserveLicenseComments: false,
	baseUrl: '.',
	paths: {
		requireLib: 'require'
	},
	name: 'index',
	out: 'index.min.js',
	include: ['requireLib']
})
```

