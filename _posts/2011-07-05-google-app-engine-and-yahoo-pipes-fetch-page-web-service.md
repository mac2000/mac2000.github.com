---
layout: post
title: Google app engine and Yahoo Pipes &#8212; fetch page web service

tags: [gae, json, pipe, python, yahoo]
---

Yahoo Pipes build in [fetch page](http://pipes.yahoo.com/pipes/docs?doc=sources#FetchPage) module have restriction - it can fetch pages only under 200 kb.

Pipe looks like this:

![screenshot](/images/wp/113.png)

But there is [web service](http://pipes.yahoo.com/pipes/docs?doc=operators#WebService) module that allow bypass this restriction.

All we need is to write web service that will fetch pages and attache them to feed.

Here is sample:

**app.yaml**

```yml
application: yahoo-pipes-fetch-page
version: 1
runtime: python
api_version: 1

handlers:
- url: /
    static_files: index.html
    upload: index.html

- url: .*
    script: main.py
```

**main.py**

```py
#!/usr/bin/env python

from google.appengine.ext import webapp
from google.appengine.ext.webapp import util

import urllib2
import re
import simplejson

class MainHandler(webapp.RequestHandler):
    def get(self):
        self.response.out.write('get: Hello world!')

class AppendHtmlHandler(webapp.RequestHandler):
    def post(self):
        data = self.request.get("data")
        obj = simplejson.loads(data)
        items = obj["items"]
        for item in items:
            req = urllib2.Request(item['link'], None, {'User-agent': 'Mozilla/5.0'})
            html = urllib2.urlopen(req).read()
            item['html'] = html[0]

        self.response.content_type = "application/json"
        simplejson.dump(obj, self.response.out)

class AppendBodyHandler(webapp.RequestHandler):
    def post(self):
        data = self.request.get("data")
        obj = simplejson.loads(data)
        items = obj["items"]
        for item in items:
            req = urllib2.Request(item['link'], None, {'User-agent': 'Mozilla/5.0'})
            html = urllib2.urlopen(req).read()
            body = re.findall(r'<body[^>]*>(.*?)</body>', html, re.DOTALL|re.MULTILINE)
            body = body[0]
            body = re.compile(r'<script.*?</script>', re.DOTALL|re.MULTILINE).sub('', body)
            body = re.compile(r'<noscript.*?</noscript>', re.DOTALL|re.MULTILINE).sub('', body)
            body = re.compile(r'<style.*?</style>', re.DOTALL|re.MULTILINE).sub('', body)
            item['body'] = body

        self.response.content_type = "application/json"
        simplejson.dump(obj, self.response.out)

def main():
    application = webapp.WSGIApplication([('/', MainHandler),
                                            ('/appendhtml', AppendHtmlHandler),
                                            ('/appendbody', AppendBodyHandler)],
                                            debug=True)
    util.run_wsgi_app(application)

if __name__ == '__main__':
    main()
```

Now you can make pipes like this:

![screenshot](/images/wp/25.png)

BUT. Here is epic fail:

```
Web service failure:
An Error Occurred
408 User-agent timeout (select)
```

So ...
