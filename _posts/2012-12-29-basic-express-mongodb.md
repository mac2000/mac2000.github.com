---
layout: post
title: Basic express mongodb todo application
tags: [nodejs, express, mongodb]
---

package.json
------------

```json
{
	"name":"todo",
	"version":"0.0.1",
	"private":true,
	"dependencies":{
		"express":"*",
		"jade":"*",
		"mongodb":"*"
	}
}
```

app.js
------

```js
var express = require('express');
var MongoClient = require('mongodb').MongoClient;
var Server = require('mongodb').Server;
var Connection = require('mongodb').Connection;
var ObjectID = require('mongodb').ObjectID;
var format = require('util').format;

var mongo_host = process.env['MONGO_NODE_DRIVER_HOST'] || 'localhost';
var mongo_port = process.env['MONGO_NODE_DRIVER_PORT'] || Connection.DEFAULT_PORT;
var mongo_db = 'test';
var mongo_url = format('mongodb://%s:%s/%s?w=1', mongo_host, mongo_port, mongo_db);
var express_port = process.env['PORT'] || 3000;

var app = express();

app.set('port', express_port);
app.set('views', __dirname + '/views');
app.set('view engine', 'jade');

app.use(express.bodyParser());
app.use(express.methodOverride());

app.get('/', function (req, res) {
	MongoClient.connect(mongo_url, function (err, db) {
		if (err) console.log(err);
		else {
			db.collection('todo').aggregate([
				{
					$project:{todo:1, status:1}
				}
			], function (err, records) {
				db.close();
				if (err) console.log(err);
				else {
					console.log(records);
					res.render('home', {title:'Home', items:records});
				}
			});
		}
	});
});

app.post('/', function (req, res) {
	var todo = req.param('todo', null);
	MongoClient.connect(mongo_url, function (err, db) {
		if (err) console.log(err);
		else {
			db.collection('todo').insert({todo:todo, created:new Date(), changed:new Date()}, function (err, records) {
				db.close();
				if (err) console.log(err);
				else {
					console.log(records);
					res.redirect('/');
				}
			});
		}
	});
});

app.put('/', function (req, res) {
	var _id = req.param('_id', null);
	var status = 'on' == req.param('status', false);
	var todo = req.param('todo', null);

	MongoClient.connect(mongo_url, function (err, db) {
		if (err) console.log(err);
		else {
			db.collection('todo').update({_id:new ObjectID(_id)}, {$set:{status:status, todo:todo, changed:new Date()}}, function (err, records_affected) {
				db.close();
				if (err) console.log(err);
				else {
					console.log(records_affected);
					res.redirect('/');
				}
			});
		}
	});
});

app.delete('/', function (req, res) {
	var _id = req.param('_id', null);

	MongoClient.connect(mongo_url, function (err, db) {
		if (err) console.log(err);
		else {
			db.collection('todo').remove({_id:new ObjectID(_id)}, function (err, records_affected) {
				db.close();
				if (err) console.log(err);
				else {
					console.log(records_affected);
					res.redirect('/');
				}
			});
		}
	});
});

app.listen(express_port);
console.log(format('Listening on http://localhost:%d', express_port));
```

views/layout.jade
-----------------

```jade
doctype 5
title= title
link(href="//netdna.bootstrapcdn.com/twitter-bootstrap/latest/css/bootstrap-combined.min.css", rel="stylesheet")
div.container-fluid(style="padding:20px")
	block content
```

views/home.jade
---------------

```jade
extends layout

block content
	h1= title

	form.form-inline(method="post", action="/")
		input(type="text", name="todo", placeholder="todo", required)
		input.btn(type="submit", value="Add")

	each item in items
		div.row-fluid
			form.form-inline(method="post", action="/", style="float:left")
				input(type="hidden", name="_method", value="put")
				input(type="hidden", name="_id", value=item._id)
				label.checkbox
					input.checkbox(type="checkbox", checked=item.status, name="status")
				input(value=item.todo, type="text", name="todo", placeholder="todo", required)
				input.btn.btn-primary(type="submit", value="Update")
			form.form-inline(method="post", action="/", style="float:left")
				input(type="hidden", name="_method", value="delete")
				input(type="hidden", name="_id", value=item._id)
				input.btn.btn-danger(type="submit", value="Delete")
```
