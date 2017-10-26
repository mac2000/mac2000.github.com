---
layout: post
title: Google apps engine + KnockoutJs + REST

tags: [gae, knockout, knockoutjs, python, rest, restful]
---

Simple example, demonstates how to create app with knockoutjs that will use
RESTfull google apps engine service.

![](/images/wp/127.png)

![](/images/wp/214.png)

**main.py:**

```py
from google.appengine.ext import webapp
from google.appengine.ext.webapp import util
from google.appengine.ext import db
from google.appengine.ext.webapp import template
from google.appengine.api import mail
from google.appengine.api import users
from django.utils import simplejson

class ServerModel(db.Model):
    name = db.StringProperty(required=True)
    active = db.BooleanProperty()
    host = db.StringProperty(required=True)

class ServerHandler(webapp.RequestHandler):
    def get(self, id = None):
        self.response.headers['Content-Type'] = 'application/json'
        servers = ServerModel.all()
        data = []
        for server in servers:
            data.append({
                'id': server.key().id(),
                'active': server.active,
                'name': server.name,
                'host': server.host,
                })

        self.response.out.write(simplejson.dumps(data))

    def post(self, id = None):
        self.response.headers['Content-Type'] = 'application/json'
        args = simplejson.loads(self.request.body)
        server = ServerModel(
            active = args.get('active'),
            name = args.get('name'),
            host = args.get('host'),
        )
        server.save()

        self.response.out.write(simplejson.dumps({
            'id': server.key().id(),
        }))

    def put(self, id = None):
        self.response.headers['Content-Type'] = 'application/json'
        args = simplejson.loads(self.request.body)
        server = ServerModel.get_by_id(args.get('id'))
        server.active = args.get('active')
        server.name = args.get('name')
        server.host = args.get('host')
        self.response.out.write('')

    def delete(self, id = None):
        self.response.headers['Content-Type'] = 'application/json'
        args = simplejson.loads(self.request.body)
        server = ServerModel.get_by_id(args.get('id'))
        server.delete()
        self.response.out.write('')

    def handle_exception(self, exception, debug_mode):
        self.error(500);
        self.response.headers['Content-Type'] = 'application/json'
        self.response.out.write(exception.message)

class MainHandler(webapp.RequestHandler):
    def get(self):
        values = {}
        self.response.out.write(template.render('templates/index.html', values))

def main():
    application = webapp.WSGIApplication([
        ('/', MainHandler),
        (r'/servers(?:/(.*))?', ServerHandler)
    ], debug=True)
    util.run_wsgi_app(application)

if __name__ == '__main__':
    main()
```

Note, that there is no validations etc, but hey this is just example.

**index.html:**

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <title>Knock</title>
    <!-- Le HTML5 shim, for IE6-8 support of HTML elements -->
    <!--[if lt IE 9]>
    <script src="http://html5shim.googlecode.com/svn/trunk/html5.js"></script>
    <![endif]-->
    <link rel="stylesheet" href="http://twitter.github.com/bootstrap/1.3.0/bootstrap.min.css">
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8">
    <script type="text/javascript" src="http://ajax.googleapis.com/ajax/libs/jquery/1.6.4/jquery.min.js"></script>
    <script type="text/javascript"
            src="http://cloud.github.com/downloads/SteveSanderson/knockout/jquery.tmpl.js"></script>
    <script type="text/javascript"
            src="http://cloud.github.com/downloads/SteveSanderson/knockout/knockout-1.2.1.js"></script>
    <script type="text/javascript" src="http://twitter.github.com/bootstrap/1.3.0/bootstrap-modal.js"></script>
    <style type="text/css">
        html, body {
            background-color: #eee;
        }

        .content {
            margin: 40px auto;
            background-color: #fff;
            padding: 20px;
            box-shadow: 0 1px 2px #ccc;
        }
    </style>
</head>
<body>
<div class="topbar">
    <div class="fill">
        <div class="container">
            <a class="brand" href="/">Knock</a>
        </div>
    </div>
</div>
<div class="container">

    <div class="content">

        <table id="servers">
            <thead>
            <tr>
                <th>Active</th>
                <th>Name</th>
                <th>Host</th>
                <th>
                    <button data-bind="click: add" data-controls-modal="serverForm" data-backdrop="true"
                    data-keyboard="true" class="btn small primary">Add server
                    </button>
                </th>
            </tr>
            </thead>
            <tbody data-bind="template: {name:'serverTemplate', foreach: servers}"></tbody>
        </table>

        <script type="text/x-jquery-tmpl" id="serverTemplate">
            <tr>
                <td data-bind="html: active"></td>
                <td data-bind="html: name"></td>
                <td data-bind="html: host"></td>
                <td>
                    <button data-bind="click: function(){ ServerForm.peak(this) }" data-controls-modal="serverForm" data-backdrop="true"
                            data-keyboard="true" class="btn small primary">Edit
                    </button>
                    &nbsp;
                    <button class="btn small secondary" data-bind="click: function(){ ServerHandler.delete(this) }">Remove</button>
                </td>
            </tr>
        </script>

        <div id="serverForm" class="modal hide fade">
            <div class="modal-header">
                <a href="#" class="close">&times;</a>

                <h3>Add/Edit serve</h3>
            </div>
            <div class="modal-body">
                <form action="">
                    <input data-bind="value: id" type="hidden" id="id" name="id"/>

                    <div class="clearfix">
                        <label for="active">Active</label>

                        <div class="input">
                            <ul class="inputs-list">
                                <li>
                                    <label>
                                        <input data-bind="checked: active" type="checkbox"
                                                value="true"
                                                name="active" id="active">
                                        <span>Check this server</span>
                                    </label>
                                </li>
                            </ul>
                        </div>
                    </div>

                    <div class="clearfix">
                        <label for="name">Name</label>

                        <div class="input">
                            <input data-bind="value: name" type="text" size="30" name="name"
                                    id="name"
                                    class="xlarge">
                        </div>
                    </div>

                    <div class="clearfix">
                        <label for="host">Host</label>

                        <div class="input">
                            <input data-bind="value: host" type="text" size="30" name="host"
                                    id="host"
                                    class="xlarge">
                        </div>
                    </div>
                </form>
            </div>
            <div class="modal-footer">
                <button data-bind="click: save" class="btn primary">Save</button>
            </div>
        </div>

    </div>
</div>

<!-- VIEW MODEL -->
<script type="text/javascript">
    var ServerHandler = {
        save: function(server) {
            var type = server.id() == 0 ? 'POST' : 'PUT';
            jQuery.ajax({
                context: server,
                data: ko.toJSON(server),
                dataTypeString: 'json',
                type: type,
                url: '/servers',
                success: function(data, textStatus, jqXHR) {
                    if(this.id() == 0) {
                        this.id(data.id);
                        ServerCollection.servers.push(this);
                    } else {
                        var server = ServerCollection.getById(this.id());
                        server.name(this.name());
                        server.active(this.active());
                        server.host(this.host());
                    }
                },
                error: function(jqXHR, textStatus, errorThrown) {
                    console.log('error',textStatus);
                    //TODO: provide app notification
                }
            });
        },
        delete: function(server) {
            jQuery.ajax({
                context: server,
                data: ko.toJSON(server),
                dataTypeString: 'json',
                type: 'DELETE',
                url: '/servers',
                success: function(data, textStatus, jqXHR) {
                    var id = this.id();
                    ServerCollection.servers.remove(function(item) { return item.id() == id });
                },
                error: function(jqXHR, textStatus, errorThrown) {
                    console.log('error',textStatus);
                    //TODO: provide app notification
                }
            });
        }
    };

    var ServerModel = function(id, name, active, host) {
        this.id = ko.observable(id);
        this.name = ko.observable(name);
        this.active = ko.observable(active);
        this.host = ko.observable(host);

        this.remove = function() {

        }

        this.save = function() {
            if (this.id() == 0) ServerCollection.items.push(this);
            console.log('ajax save for server');
        }

        this.fillForm = function() {
            serverForm.id(this.id());
            serverForm.name(this.name());
            serverForm.active(this.active());
            serverForm.host(this.host());
        }
    }

    var ServerCollection = {
        servers: ko.observableArray([]),

        getById: function(id) {
            for (i = 0;  i < this.servers().length; i++)
                if (this.servers()[i].id() == id) return this.servers()[i];
            return false;
        },

        add: function() {
            ServerForm.peak(new ServerModel(0, '', true, ''));
        },

        read: function() {
            jQuery.ajax({
                dataTypeString: 'json',
                type: 'GET',
                url: '/servers',
                success: function(data, textStatus, jqXHR) {
                    jQuery.each(data, function(index, item){
                        ServerCollection.servers.push(new ServerModel(item.id, item.name, item.active, item.host));
                    });
                },
                error: function(jqXHR, textStatus, errorThrown) {
                    console.log('error',textStatus);
                    //TODO: provide app notification
                }
            });
        }
    }
    ko.applyBindings(ServerCollection, $('#servers').get(0));
    ServerCollection.read();

    var ServerForm = {
        id: ko.observable(0),
        name: ko.observable(''),
        active: ko.observable(false),
        host: ko.observable(''),

        peak: function(server) {
            this.id(server.id());
            this.name(server.name());
            this.active(server.active());
            this.host(server.host());
        },

        save: function() {
            ServerHandler.save(new ServerModel(this.id(), this.name(), this.active(), this.host()));
            $('#serverForm').modal('hide');
        }
    }

    ko.applyBindings(ServerForm, $('#serverForm').get(0));
</script>

</body>
</html>
```

Note that for simplify, there is two models rather that just one like in
knockoutjs examples, but u can do whatever way u want, also there is no
validations etc.
