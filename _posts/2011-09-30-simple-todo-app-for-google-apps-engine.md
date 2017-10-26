---
layout: post
title: Simple TODO app for google apps engine

tags: [db, example, gae, python]
---

This is copy-paste from:

http://www.vogella.de/articles/GoogleAppEngine/article.html

Screenshot:

![screenshot](/images/wp/126.png)

**app.yaml**

```yml
application: todo
version: 1
runtime: python
api_version: 1

handlers:
- url: /css
    static_dir: css
- url: /images
    static_dir: images
- url: .*
    script: main.py
```

**main.py**

```py
#!/usr/bin/env python
#
# Copyright 2007 Google Inc.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#
from google.appengine.ext import webapp
from google.appengine.ext.webapp import util
from google.appengine.ext import db
from google.appengine.ext.webapp import template
from google.appengine.api import mail
from google.appengine.api import users

class TodoModel(db.Model):
    author = db.UserProperty(required=True)
    shortDescription = db.StringProperty(required=True)
    longDescription = db.StringProperty(multiline=True)
    url = db.StringProperty()
    created = db.DateTimeProperty(auto_now_add=True)
    updated = db.DateTimeProperty(auto_now=True)
    dueDate = db.StringProperty(required=True)
    finished = db.BooleanProperty()

class MainHandler(webapp.RequestHandler):
    def get(self):
        user = users.get_current_user()
        url = users.create_login_url(self.request.uri)
        url_linktext = 'Login'

        if user:
            url = users.create_logout_url(self.request.uri)
            url_linktext = 'Logout'

        todos = TodoModel.gql("WHERE author = :author and finished=false",
                                author=users.get_current_user())

        values = {
            'todos': todos,
            'numbertodos': todos.count(),
            'user': user,
            'url': url,
            'url_linktext': url_linktext,
            }
        self.response.out.write(template.render('templates/index.html', values))

class New(webapp.RequestHandler):
    def post(self):
        user = users.get_current_user()
        if user:
            testurl = self.request.get('url')
            if not testurl.startswith('http://') and testurl:
                testurl = 'http://' + testurl

            todo = TodoModel(
                author=users.get_current_user(),
                shortDescription=self.request.get('shortDescription'),
                longDescription=self.request.get('longDescription'),
                dueDate=self.request.get('dueDate'),
                url=testurl,
                finished=False
            )
            todo.put()

        self.redirect('/')

class Email(webapp.RequestHandler):
    def get(self):
        user = users.get_current_user()
        if user:
            raw_id = self.request.get('id')
            id = int(raw_id)
            todo = TodoModel.get_by_id(id)
            message = mail.EmailMessage(
                sender=user.email(),
                subject=todo.shortDescription
            )
            message.to = user.email()
            message.body = todo.longDescription
            message.send()

        self.redirect('/')

class Done(webapp.RequestHandler):
    def get(self):
        user = users.get_current_user()
        if user:
            raw_id = self.request.get('id')
            id = int(raw_id)
            todo = TodoModel.get_by_id(id)
            todo.delete()
        self.redirect('/')

def main():
    application = webapp.WSGIApplication([
        ('/', MainHandler),
        ('/new', New),
        ('/done', Done),
        ('/email', Email)
    ], debug=True)
    util.run_wsgi_app(application)

if __name__ == '__main__':
    main()
```

**templates/index.html**

```html
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN"
        "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html>
<head>
    <title>Todos</title>
    <link rel="stylesheet" href="http://twitter.github.com/bootstrap/1.3.0/bootstrap.min.css">
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8"/>
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

        form, fieldset, div.actions {
            margin-bottom: 0
        }
    </style>
</head>
<body>
<div class="topbar">
    <div class="fill">
        <div class="container">
            <a class="brand" href="/">//TODO: {{user.nickname}}</a>

            <div class="pull-right">
                <ul class="nav">
                    <li class="active"><a href="{{ url }}">{{ url_linktext }}</a></li>
                </ul>
            </div>
        </div>
    </div>
</div>
<div class="container">

    <div class="content">

        {% if user %}

        {% if numbertodos %}

        You have a total number of {{numbertodos}} Todos.

        <table class="sortable zebra-striped">
            <tr>
                <th>Short description</th>
                <th>Due Date</th>
                <th>Long Description</th>
                <th>URL</th>
                <th>Created</th>
                <th>Updated</th>
                <th>Done</th>
                <th>Send Email reminder</th>
            </tr>

            {% for todo in todos %}
            <tr>
                <td>
                    {{todo.shortDescription}}
                </td>
                <td>
                    {{todo.dueDate}}
                </td>
                <td>
                    {{todo.longDescription}}
                </td>
                <td>
                    {% if todo.url %}
                    <a href="{{todo.url}}"> {{todo.url}}</a>

                    {% endif %}
                </td>
                <td>
                    {{todo.created|date:"d.m.Y"}}
                </td>
                <td>
                    {{todo.updated|date:"d.m.Y"}}
                </td>
                <td>
                    <a class="done" href="/done?id={{todo.key.id}}">Done</a>
                </td>
                <td>
                    <a class="email" href="/email?id={{todo.key.id}}">Email</a>
                </td>
            </tr>
            {% endfor %}
        </table>

        {% else %}

        <div class="alert-message block-message warning">
            <p>There is no TODOs for <strong>{{user.nickname}}</strong>.</p>
        </div>

        {% endif %}

        <form action="/new" method="post">
            <fieldset>
                <legend>Create new Todo</legend>

                <div class="clearfix">
                    <label for="shortDescription">Summary</label>

                    <div class="input">
                        <input class="xxlarge" id="shortDescription" name="shortDescription" size="30" type="text">
                    </div>
                </div>

                <div class="clearfix">
                    <label for="dueDate">Due Date</label>

                    <div class="input">
                        <input class="xxlarge" id="dueDate" name="dueDate" size="30" type="text">
                    </div>
                </div>

                <div class="clearfix">
                    <label for="longDescription">Description</label>

                    <div class="input">
                        <textarea class="xxlarge" id="longDescription" name="longDescription" rows="3"></textarea>
                    </div>
                </div>

                <div class="clearfix">
                    <label for="url">URL</label>

                    <div class="input">
                        <input class="xxlarge" id="url" name="url" size="30" type="text">
                    </div>
                </div>

                <div class="actions">
                    <input type="submit" class="btn primary" value="Save">&nbsp;
                    <button type="reset" class="btn">Cancel</button>
                </div>
            </fieldset>
        </form>

        {% else %}
        <div class="alert-message block-message info">
            <p>To create\edit TODOs you need <a href="{{ url }}">login</a> with your Google account.</p>
        </div>
        {% endif %}

    </div>
</div>
</body>
</html>
```

## Ajax \ Rest like implementations

![screenshot](/images/wp/213.png)

**app.yaml**

```yml
application: ajaxmodel
version: 1
runtime: python
api_version: 1

handlers:
- url: .*
    script: main.py
```

**main.py**

```py
#!/usr/bin/env python
#
# Copyright 2007 Google Inc.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#

import cgi
from google.appengine.ext import webapp
from google.appengine.ext.webapp import util
from google.appengine.ext import db
from google.appengine.ext.webapp import template
from google.appengine.api import mail
from google.appengine.api import users
from django.utils import simplejson

class TaskModel(db.Model):
    title = db.StringProperty(required=True)
    isDone = db.BooleanProperty()

class TaskHandler(webapp.RequestHandler):
    def get(self):
        tasks = TaskModel.all()
        data = []
        for task in tasks:
            data.append({
                'id': task.key().id(),
                'title': task.title,
                'isDone': task.isDone,
                })

        self.response.out.write(simplejson.dumps({
            'success': True,
            'message': 'ok',
            'data': data
        }))

    def post(self):
        task = TaskModel(
            title=self.request.get('title'),
            isDone=self.request.get('isDone').upper() == 'TRUE',
        )
        task.put()

        self.response.out.write(simplejson.dumps({
            'success': True,
            'message': 'ok',
            'data': []
        }))

    def put(self):
        params = dict([part.split('=') for part in self.request.body.split('&')])
        id = int(params.get('id'))
        isDone = False
        if params.get('isDone').upper() == 'TRUE':
            isDone = True

        task = TaskModel.get_by_id(id)
        task.isDone = isDone
        task.title = params.get('title')
        task.save()

        self.response.out.write(simplejson.dumps({
            'success': True,
            'message': 'ok',
            'data': []
        }))

    def delete(self):
        params = dict([part.split('=') for part in self.request.body.split('&')])
        id = int(params.get('id'))
        task = TaskModel.get_by_id(id)
        task.delete()

        self.response.out.write(simplejson.dumps({
            'success': True,
            'message': 'ok',
            'data': []
        }))

    def handle_exception(self, exception, debug_mode):
        self.response.out.write(simplejson.dumps({
            'success': False,
            'message': exception.message,
            'data': []
        }))

"""
class TaskHandler(webapp.RequestHandler):
    def get(self, id_raw = None):
        self.response.out.write('hello : ' + str(id_raw))

    def post(self, id_raw = None):
        pass
"""

class MainHandler(webapp.RequestHandler):
    def get(self):
        values = {
            'login_url': users.create_login_url(self.request.uri),
            'logout_url': users.create_logout_url(self.request.uri),
            }
        self.response.out.write(template.render('templates/index.html', values))

def main():
    application = webapp.WSGIApplication([
        ('/', MainHandler),
        #(r'/api/task(?:/(.*))?', TaskHandler)
        ('/tasks', TaskHandler)
    ], debug=True)
    util.run_wsgi_app(application)

if __name__ == '__main__':
    main()
```

**index.html**

```html
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN"
        "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html>
<head>
    <title>Todos</title>
    <link rel="stylesheet" href="http://twitter.github.com/bootstrap/1.3.0/bootstrap.min.css">
    <meta http-equiv="Content-Type" content="text/html; charset=UTF-8"/>
    <script type="text/javascript" src="http://ajax.googleapis.com/ajax/libs/jquery/1.6.4/jquery.min.js"></script>
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
            <a class="brand" href="/">Tasks</a>
        </div>
    </div>
</div>
<div class="container">

    <div class="content">

        <form>
            <fieldset>
                <legend>Add task</legend>
                <div class="clearfix">
                    <label for="newTask">Task</label>

                    <div class="input">
                        <input id="newTask" name="newTask" placeholder="What needs to be done?" class="xxlarge"
                                size="30"
                                type="text"/>
                    </div>
                </div>
                <div class="actions">
                    <input id="addTask" type="submit" class="btn primary" value="Add">
                </div>
            </fieldset>
        </form>

        <table id="tasks" class="zebra-striped"></table>

    </div>
</div>
<script type="text/javascript">
    function refresh() {
        $.ajax({
            url: '/tasks',
            type: 'GET',
            dataType: 'json',
            success: function(response) {
                if (response.success == true) {
                    $('#tasks').empty();
                    if (response.data && response.data.length > 0) {
                        $('#tasks').append('<tr><th>Id</th><th>Is done</th><th>Task</th><th></th></tr>');
                    }
                    $.each(response.data, function(index, item) {
                        var isDone = item.isDone ? 'Yes' : 'No';

                        $('#tasks').append('<tr><td>' + item.id + '</td><td><a href=="#" onclick="editTask(' + item.id + ', ' + !item.isDone + ', \'' + item.title + '\');return false;">' + isDone + '</a></td><td><a href=="#" onclick="editTask(' + item.id + ', ' + item.isDone + ');return false;">' + item.title + '</a></td><td><a href="#" onclick="deleteTask(' + item.id + ');return false;">Delete</a></td></tr>');
                    });
                }
                else {
                    alert(response.message); //TODO: notify
                }
            },
            error: function(jqXHR, textStatus, errorThrown) {
                alert('err') //TODO: notify
            }
        });
    }

    function editTask(id, isDone, title) {
        var newTitle = (typeof title == 'undefined') ? prompt('Enter new title') : title;

        $.ajax({
            url: '/tasks',
            type: 'PUT',
            dataType: 'json',
            data: {id: id, isDone: isDone, title: newTitle},
            success: function(response) {
                if (response.success == true) {
                    refresh();
                }
                else {
                    alert(response.message); //TODO: notify
                }
            },
            error: function(jqXHR, textStatus, errorThrown) {
                alert('err') //TODO: notify
            }
        });
    }

    function deleteTask(id) {
        if(!confirm('Confirm deletion')) return;

        $.ajax({
            url: '/tasks',
            type: 'DELETE',
            dataType: 'json',
            data: {id: id},
            success: function(response) {
                if (response.success == true) {
                    refresh();
                }
                else {
                    alert(response.message); //TODO: notify
                }
            },
            error: function(jqXHR, textStatus, errorThrown) {
                alert('err') //TODO: notify
            }
        });
    }

    jQuery(document).ready(function($) {
        $('#addTask').click(function(e) {
            e.preventDefault();
            $.ajax({
                url: '/tasks',
                type: 'POST',
                dataType: 'json',
                data: { title: $('#newTask').val(), isDone: false },
                success: function(response) {
                    console.log(response);
                    if (response.success == true) {
                        refresh();
                    }
                    else {
                        alert(response.message); //TODO: notify
                    }
                },
                error: function(jqXHR, textStatus, errorThrown) {
                    alert('err') //TODO: notify
                }
            });
            return false;
        });

        refresh();
    });
</script>
</body>
</html>
```

TODO: implement REST by refs & use knockout
